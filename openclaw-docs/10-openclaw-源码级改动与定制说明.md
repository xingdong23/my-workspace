# 第 10 篇：OpenClaw 源码级改动与定制说明

这一篇不讲配置，而是讲一个更现实的问题：

**当配置已经不够用时，OpenClaw 应该怎么改源码？**

很多人装好 OpenClaw 后，最先遇到的不是“不会配置”，而是：
- 飞书共享群里，多个 bot 的提及路由不符合预期
- 某些 OpenAI 兼容代理，小请求能通，但真实 agent 请求被拦
- 本地代理环境下，`web_fetch` 明明访问的是公网网址，却被 SSRF 防护挡住

这些问题往往已经超出 `openclaw.json` 能解决的范围。  
这时候就要进入源码级定制。

---

## 1. 什么情况下应该改源码

只有在下面三种情况下，才建议动源码：

1. **问题根因在插件适配层**  
例如飞书消息事件已经进来了，但路由语义不对。

2. **问题根因在 provider 兼容层**  
例如 API 端点、模型名、Key 都没问题，但 OpenClaw 的真实请求形态被上游拦截。

3. **问题根因在安全或网络策略层**  
例如本地代理把公网域名解析成 fake-ip，导致 web tools 被误判为访问内网。

如果问题只是 prompt 不够清楚、配置字段没配对、session 太胖、skills 不合理，那优先还是回到配置和工作区层面解决，不要一上来就改源码。

---

## 2. OpenClaw 的源码到底在哪里

如果你是用 Node 全局安装，或者通过 Homebrew 入口启动，真正运行的 OpenClaw 通常还是在全局包目录里。

典型路径是：

```text
/opt/homebrew/lib/node_modules/openclaw
```

关键分成两块：

- `dist/`
  - 主程序打包后的运行时代码
- `extensions/`
  - 各渠道插件源码，例如 Feishu

最常见的两个定制点就是：

```text
/opt/homebrew/lib/node_modules/openclaw/dist/
/opt/homebrew/lib/node_modules/openclaw/extensions/feishu/src/
```

也就是说：

- 主程序运行逻辑，多半在 `dist/`
- 飞书这类渠道适配，多半在 `extensions/feishu/src/`

---

## 3. 定制场景一：Feishu 多 bot 提及路由

### 典型问题

在一个共享群里：
- 人类明明 `@ceo`
- `trade` 却也跑出来抢答
- 或者 `@所有人` 时，行为和预期不一致

这种问题很多人第一反应是“模型幻觉”。

但如果你查过日志和 transcript，经常会发现：
- 消息确实收到了
- 问题不在模型
- 而在飞书插件如何解析 mention、如何构造入站上下文、如何做路由判断

### 合理的定制目标

如果你要把共享群做成长期可用的多 bot 协作环境，最起码应该满足：

- `@ceo` 时，未被点名的 bot 不抢答
- `@ceo @trade` 时，被点名的 bot 都能进入判断
- `@所有人` / `@_all` 时，语义上等价于“显式点名全部已配置 bot”

### 最值得改的点

Feishu 插件里最关键的是这几段：

- mention 判断  
  [/opt/homebrew/lib/node_modules/openclaw/extensions/feishu/src/bot.ts](/opt/homebrew/lib/node_modules/openclaw/extensions/feishu/src/bot.ts)
- 上下文类型定义  
  [/opt/homebrew/lib/node_modules/openclaw/extensions/feishu/src/types.ts](/opt/homebrew/lib/node_modules/openclaw/extensions/feishu/src/types.ts)

实际可行的改法是：

1. 保留“消息里到底点名了谁”
2. 给入站上下文增加 `mentionedConfiguredBots`
3. 如果消息明确点名了别的已配置 bot，而当前 bot 没被点到，就直接跳过
4. 把 `@_all` 展开成当前所有已配置 bot

### 改完后的效果

这样做的价值不是“让模型更聪明”，而是：

- 路由层先把不该进来的 bot 挡掉
- 进入模型前就把“点名信息”保住
- 避免一切都靠 prompt 自己猜

这类问题本质是渠道适配问题，不是 prompt 设计问题。

---

## 4. 定制场景二：OpenAI 兼容代理的请求指纹问题

### 典型问题

这类问题非常迷惑人：

- `curl` 最小请求正常
- `/v1/models` 正常
- Key 也正常
- 模型名也存在
- 但只要真正走 OpenClaw agent，就报：

```text
403 Your request was blocked.
```

### 这类问题应该怎么判断

不能只看：
- “小请求通不通”
- “Node fetch 能不能通”
- “模型列表是不是有”

真正要看的是：

- OpenClaw 真实发出去的请求 body
- OpenClaw 真实带了哪些 headers
- 和你手写请求相比，差异到底在哪里

### 一类非常典型的根因

有些 OpenAI 兼容代理站，会对特定 SDK 指纹做拦截。  
最常见的表现就是：

- 普通手写请求没问题
- 但带了 SDK 特征头之后就被风控

这时候需要改的不是模型配置，而是运行时代码里的出站请求包装层。

### 合理的定制方式

在这种场景下，最稳的做法不是全局乱改，而是：

- 只对特定域名做定向兼容
- 只移除明确触发风控的请求头
- 不影响其他 provider

这类改动通常会落在：

```text
/opt/homebrew/lib/node_modules/openclaw/dist/index.js
```

### 这类补丁的定位

这不是“业务逻辑改动”，而是**provider 兼容层补丁**。  
目标只有一个：让 OpenClaw 的真实 agent 请求形态，能被该代理正常接受。

---

## 5. 定制场景三：本地代理、fake-ip 和 web tools 的冲突

### 典型问题

你在本机开了 Clash、Mihomo、Surge 这类代理后，OpenClaw 里可能出现这种报错：

```text
Blocked: resolves to private/internal/special-use IP address
```

但你访问的明明是公网网址，比如：
- Yahoo Finance
- TradingView
- 微信文章
- 文档站点

### 为什么会这样

因为一些代理软件会用 fake-ip 模式，把公网域名先解析到类似：

```text
198.18.0.x
```

OpenClaw 的 SSRF 防护只看到：
- 这个域名解析结果不像公网
- 更像 private/internal/special-use 地址

于是直接把请求拦掉。

### 这里到底应该改哪一层

理论上有两种方向：

1. 改代理配置  
让这些域名不要走 fake-ip

2. 改 OpenClaw 的 web tools SSRF 策略  
让 trusted-network 场景下允许这类 fake-ip 解析

如果你是在自己的本机环境里用，并且明确知道为什么要放宽，这时可以做第二种定制。

### 这类改动通常在哪

一般会落在：

```text
/opt/homebrew/lib/node_modules/openclaw/dist/subagent-registry-*.js
```

本质上是给 `web_fetch` / `web_search` 这条链路补上更宽松的 `ssrfPolicy`。

### 这类补丁的性质

这类补丁不是功能增强，而是**安全策略放宽**。  
所以它一定要写进文档里，不能改了就忘。

---

## 6. 源码定制时最重要的原则

真正成熟的改法，不是“大改”，而是“小而准”。

我建议遵守这 5 条：

1. **先拿证据，再下手**  
先看日志、session transcript、真实请求，不要靠猜。

2. **只改最小闭环**  
能改一处解决，就不要改三处。

3. **优先做定向补丁**  
例如只对某个 provider、某个渠道、某种场景生效。

4. **必须有回归验证**  
至少要有一条最小测试，或者一条稳定的复测路径。

5. **把补丁写进文档**  
以后升级后行为变了，第一时间就能知道是不是补丁丢了。

---

## 7. 改完之后怎么验证

源码补丁如果没有验证，等于没改。

最起码要做三类验证：

### 路由类补丁
- 看日志里有没有进入正确路由
- 看 transcript 里 agent 实际看到的内容是不是对的
- 看未被点名的 bot 有没有被挡掉

### Provider 兼容补丁
- 测最小 agent 请求
- 测真实默认会话
- 确认不是只有 `curl` 通，而是 OpenClaw 真的通

### 安全策略补丁
- 用目标站点复测
- 确认以前被拦的 URL 现在能过
- 确认不是把全局安全都关掉了

---

## 8. 最大的维护风险：升级覆盖

只要你改的是安装目录：

```text
/opt/homebrew/lib/node_modules/openclaw/...
```

那就要默认下面这些动作可能把补丁覆盖掉：

- `openclaw update`
- 重新安装 OpenClaw
- Node 全局包重装

所以成熟一点的做法是：

1. 先允许自己做“热修”
2. 但最后一定要把热修沉淀成：
   - patch 文件
   - 文档说明
   - 或自己的覆盖插件

否则你下次升级之后，很容易又回到老问题。

---

## 9. 教程角度的最终建议

如果你准备写给别人看，这一章最应该传达的不是“我改了哪些文件”，而是这三件事：

1. **什么时候应该改源码**
2. **源码该改哪一层**
3. **改完如何验证、如何维护**

所以从教程角度，OpenClaw 的源码定制最值得总结成下面三类案例：

- 渠道适配层：Feishu 多 bot 提及与共享群路由
- Provider 兼容层：OpenAI 兼容代理的请求指纹问题
- 安全策略层：本地代理、fake-ip 与 web tools 的 SSRF 冲突

这三类场景足够覆盖大多数“配置已经不够用”的高级问题。

---

## 10. 一句话总结

源码定制不是 OpenClaw 的日常使用方式，但它是 OpenClaw 真正强大的地方。

你不需要一开始就改源码。  
但当你已经确认：

- 根因不在 prompt
- 根因不在配置
- 根因不在 session

那就应该敢于进入插件层、provider 层和安全策略层，做最小、可验证、可回滚的定制。
