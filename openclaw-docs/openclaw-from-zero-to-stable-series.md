# 把 OpenClaw 从 0 跑到稳定：一套从安装、飞书接入、多机器人协作到源码修补的完整实战手册

## 系列定位

这不是“5 分钟装机成功”的演示稿，而是一套真正把 OpenClaw 跑起来、跑稳、跑清爽之后的经验总结。

目标读者有两类：

- 想用 OpenClaw 搭自己的 AI 工作台的新手
- 已经装上了，但开始遇到飞书、多 agent、会话、升级、插件问题的进阶用户

这套文章的核心目标不是“让你装上”，而是“让你长期可维护”。

---

## 第 1 章：先别急着装，先理解 OpenClaw 到底是什么

很多人第一次接触 OpenClaw，会把它理解成“一个 AI 机器人框架”。

但我真正把它接上飞书、拉起多 agent、修掉群聊问题之后，发现它更准确的定位是：

**OpenClaw 不是一个机器人，而是一套本地 AI 操作系统。**

它至少同时管理 5 层东西：

1. 模型层
2. agent 层
3. workspace 层
4. channel 层
5. plugin/skill 层

也就是说，它不是“配置一个 bot 就结束”，而是“你在本地维护一套 AI 基础设施”。

为什么这点要先讲清楚？

因为如果你把它当成普通聊天机器人来理解，后面遇到下面这些问题时会很崩：

- 为什么配置对了还不回复
- 为什么 dashboard 看到一堆旧会话
- 为什么同一个群里多个 bot 会抢答
- 为什么更新之后某些行为变了
- 为什么有时候甚至要改插件源码

这不是 OpenClaw 不好，而是它本来就不是“纯前台产品”。

---

## 第 2 章：从 0 安装 OpenClaw，先用最稳的方式

我最终采用的是最简单也最稳的一种方式：Node 全局安装。

```bash
brew install node
npm install -g openclaw
```

安装完先不要急着配飞书，先确认 CLI 本身通了：

```bash
openclaw --version
openclaw doctor
openclaw status
```

我这边最终确认到的安装形态是：

- 命令入口：`/opt/homebrew/bin/openclaw`
- 实际包目录：`/opt/homebrew/lib/node_modules/openclaw`

这两个路径非常重要，因为后面你排查插件、看版本、甚至改源码，都会用到。

安装后的第一条建议是：

```bash
openclaw update
openclaw doctor --fix
```

因为 OpenClaw 的配置字段和 channel 结构会演进，升级之后如果不做 `doctor --fix`，很容易出现“版本是新的，但配置还是旧结构”的情况。

---

## 第 3 章：不要一开始就上多机器人，先跑通一个最小系统

我非常建议先跑通一个单 agent，再扩展成多 agent。

原因很简单：如果你一开始就同时引入模型、飞书、多 workspace、多账号、多心跳，出了问题根本不知道是哪个层面错了。

一个最小可用的 OpenClaw 配置至少要有这几部分：

- 一个 provider
- 一个 agent
- 一个 workspace
- 一个 channel
- 一个 binding

一个最小思路可以是这样：

```json
{
  "models": {
    "providers": {
      "zai": {
        "baseUrl": "https://open.bigmodel.cn/api/coding/paas/v4",
        "apiKey": "<YOUR_API_KEY>",
        "api": "openai-completions",
        "models": [
          {
            "id": "glm-5",
            "name": "glm-5",
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "list": [
      {
        "id": "ceo",
        "default": true,
        "workspace": "/Users/yourname/.openclaw/workspace-ceo",
        "agentDir": "/Users/yourname/.openclaw/agents/ceo/agent"
      }
    ]
  },
  "bindings": [
    {
      "agentId": "ceo",
      "match": {
        "channel": "feishu",
        "accountId": "ceo"
      }
    }
  ]
}
```

这里最重要的原则不是字段多少，而是**命名对齐**：

- agent id 用 `ceo`
- workspace 用 `workspace-ceo`
- accountId 也尽量叫 `ceo`

只要一开始命名收口，后面维护成本会低很多。

---

## 第 4 章：飞书接入，真正的难点不是填 App ID 和 Secret

很多教程会把飞书接入写得很轻松，像是：

1. 创建应用
2. 填 `appId`
3. 填 `appSecret`
4. 结束

真实情况远没有这么简单。

飞书接入的难点在于：

- 权限有没有真正生效
- 事件有没有真正投递给 OpenClaw
- 网关是不是已经起来了
- 你看到的“不回复”，到底是模型问题，还是事件问题

飞书接入完成后，我建议固定做这几步验证：

```bash
openclaw config validate
openclaw doctor
openclaw gateway restart
openclaw gateway status
openclaw plugins list
openclaw status
```

这几条命令是飞书排障的基本盘。

一个最基本的飞书配置长这样：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "defaultAccount": "ceo",
      "groupPolicy": "open",
      "requireMention": false,
      "streaming": false,
      "blockStreaming": true,
      "accounts": {
        "ceo": {
          "enabled": true,
          "appId": "<APP_ID>",
          "appSecret": "<APP_SECRET>"
        }
      }
    }
  }
}
```

这里有两个经验值：

- `streaming: false` 在飞书里通常更稳
- `blockStreaming: true` 能减少分段输出带来的群聊噪音

---

## 第 5 章：Skills 才是 OpenClaw 真正的能力放大器

如果前面几章解决的是“装起来、接起来、跑起来”，那这一章要讲的是另一件经常被低估的事：

**没有 skills 的 OpenClaw，其实很笨。**

它当然也能聊天，也能回消息，但很多时候只是“会说”，谈不上“会做”。

### 1. Skills 到底是什么

你可以把 skills 理解成：

- 能力包
- 工作流模板
- 某一类任务的操作说明书
- 让 agent 在特定领域里更会做事的本地能力模块

它不是简单的 prompt 收藏夹，也不只是几段角色设定。

真正有用的 skill，通常会包含：

- 触发条件
- 工作步骤
- 工具使用建议
- 输出要求
- 约束边界

所以从实际效果上看：

- 没有 skills 的 OpenClaw：更像一个会聊天的模型壳
- 有了 skills 的 OpenClaw：更像一个能执行工作流的 agent 系统

### 2. 为什么我会说“没有 skills 的 OpenClaw 很傻”

因为很多“机器人怎么这么笨”的问题，本质上都不是模型差，而是：

- 它没有明确的工作路径
- 它不知道这类任务该怎么下手
- 它缺领域能力
- 它只能靠临场猜

比如同样是做事情：

- 没有 skill，它只能泛泛回答
- 有了 skill，它知道先查什么、再做什么、最后怎么输出

这就是为什么 skills 对 OpenClaw 不是附属品，而是能力放大器。

### 3. Skills 和插件不是一回事

插件更偏“接口层”：

- 飞书
- Discord
- Telegram
- 各类 provider 接入

Skills 更偏“能力层”：

- 某类问题怎么做
- 某类任务走什么流程
- 某个 agent 在某种场景下如何更强

所以插件让 OpenClaw 能接世界，skills 让 OpenClaw 真能做事。

### 4. 如何查看和安装 skills

最直接的方法是：

```bash
openclaw skills list
openclaw skills install <skill-name>
```

比如：

```bash
openclaw skills install feishu-doc
openclaw skills install feishu-drive
openclaw skills install copywriting
openclaw skills install github
```

装完之后，不要直接假设已经能用了。至少再做两步：

```bash
openclaw skills list
openclaw status
```

重点不是“有没有出现在列表里”，而是它是不是 `ready`。

### 5. 我建议优先关注的几类常用 skills

如果你是围绕飞书和多 agent 在用 OpenClaw，我认为最值得优先理解的几类是：

- 飞书生态类：`feishu-doc`、`feishu-drive`、`feishu-perm`、`feishu-wiki`
- 搜索和信息类：`agent-reach`、`find-skills`、`blogwatcher`、`daily-ai-news`
- 写作和内容类：`copywriting`、`content-strategy`、`ai-daily-digest`
- 技术和代码类：`coding-agent`、`github`、`gh-issues`

### 6. Skills 不是越多越好

更好的做法是：

- 先围绕你的高频场景装
- 每个 agent 围绕自己的职责装核心 skills
- 常用的保留，不常用的先别堆太多

比如：

- `writer` 更适合配写作类、内容类、飞书文档类
- `trade` 更适合配技术类、执行类
- `ceo` 更适合配分析类、协调类、信息整合类

这样 skill 是在放大角色，而不是把所有 bot 搅成一团。

### 7. 我对 skills 的最终判断

如果把 OpenClaw 比作一台机器：

- provider 决定大脑
- channel 决定接口
- workspace 决定身份和记忆
- skills 决定它到底会不会干活

所以我会把 skills 放在一个很高的位置：

**OpenClaw 能不能真正有生产力，skills 至少占一半。**

---

## 第 6 章：从单机器人扩展到多机器人，重点不是复制目录，而是系统设计

我最后把系统扩展成了四个角色：

- `ceo`：产品、运营、分析、协调
- `trade`：技术、交易、系统执行
- `writer`：写文章、文稿整理、内容输出
- `crypto`：加密货币研究和辅助判断

这一步最容易踩的坑是：

**表面上是新增两个机器人，实际上是新增两套运行态系统。**

你不仅要新增 workspace，还要同步新增：

- `agents.list`
- `agentDir`
- `bindings`
- `channels.feishu.accounts`
- 对应的 `memory`、`heartbeat`、会话目录

我最后保留下来的核心结构如下：

```json
{
  "agents": {
    "list": [
      {
        "id": "trade",
        "workspace": "/Users/yourname/.openclaw/workspace-trade",
        "agentDir": "/Users/yourname/.openclaw/agents/trade/agent",
        "heartbeat": { "every": "30m" }
      },
      {
        "id": "ceo",
        "default": true,
        "workspace": "/Users/yourname/.openclaw/workspace-ceo",
        "agentDir": "/Users/yourname/.openclaw/agents/ceo/agent",
        "heartbeat": { "every": "30m" }
      },
      {
        "id": "writer",
        "workspace": "/Users/yourname/.openclaw/workspace-writer",
        "agentDir": "/Users/yourname/.openclaw/agents/writer/agent",
        "heartbeat": { "every": "30m" }
      },
      {
        "id": "crypto",
        "workspace": "/Users/yourname/.openclaw/workspace-crypto",
        "agentDir": "/Users/yourname/.openclaw/agents/crypto/agent",
        "heartbeat": { "every": "30m" }
      }
    ]
  }
}
```

我的建议是：

**不要整目录无脑复制。**

可以复制结构，但不要继承这些东西：

- 旧 session
- 旧 memory
- 旧 `.git`
- 旧退役业务文档
- 旧 open_id 说明文件

新增 agent 最好是“复制骨架，清空状态”。

---

## 第 7 章：多 agent 协作要靠内部 handoff，不要只靠群里互相 @

这是我这次最重要的认知之一。

直觉上，你会觉得多机器人在同一个飞书群里，应该可以这样工作：

1. 人类 `@ceo`
2. `ceo` 群里再 `@trade`
3. `trade` 接着说

但实际运行后你会发现，这条链路并不可靠。

原因在于：**飞书的消息展示层和机器人入站事件层，不是一回事。**

人类当然能看到 bot 发的消息。

但另一个 bot 不一定会收到“这条 bot 消息”的正式入站事件。

所以真正稳定的做法不是让 bot 在群里互相喊话，而是：

- 人类在群里 `@ceo`
- `ceo` 在 OpenClaw 内部把任务交给 `trade`
- `trade` 在后台完成
- `ceo` 再对外回复

这就是我后来采用的思路：

**群里对人展示，内部做真正协作。**

对应配置上，需要打开 agent-to-agent：

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["trade", "ceo", "writer", "crypto"]
    }
  },
  "session": {
    "agentToAgent": {
      "maxPingPongTurns": 3
    }
  }
}
```

这一步的价值非常大，因为它让你的多 agent 系统不依赖渠道本身的奇怪行为。

---

## 第 8 章：共享群策略是 OpenClaw 在飞书里最容易踩坑的地方

如果多个机器人长期驻留在同一个飞书群里，最难的不是接入，而是**行为控制**。

你需要回答四个问题：

1. 群里所有消息，机器人都看吗
2. 不 `@` 也能回复吗
3. 回复时要不要引用原消息
4. 要不要沿着旧 thread 继续回

我最终采用的共享群配置思路是：

- 群内允许观察
- 但只有“被点名、能提供增量信息、或能推动决策”才回复
- 回复时只引用当前消息一次
- 不沿着历史 thread 往下钻

配置大概是这样：

```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "open",
      "accounts": {
        "ceo": {
          "requireMention": false,
          "groups": {
            "oc_xxx_shared_group": {
              "replyToMode": "first",
              "replyInThread": "disabled",
              "systemPrompt": "群聊参与规则：只有在被明确点名、能提供增量信息、或能推动决策时才回复；否则输出 NO_REPLY。"
            }
          }
        },
        "trade": {
          "requireMention": false,
          "groups": {
            "oc_xxx_shared_group": {
              "replyToMode": "first",
              "replyInThread": "disabled",
              "systemPrompt": "与 ceo 使用相同的群聊参与规则。"
            }
          }
        }
      }
    }
  }
}
```

这里两个关键字段的实际意义是：

- `replyToMode: "first"`
  回复时引用一次原消息，清楚但不刷屏

- `replyInThread: "disabled"`
  不要掉进飞书历史话题链，不然会出现“机器人总在很旧的消息下面回复”的怪现象

---

## 第 9 章：运维才是 OpenClaw 真正的门槛

安装好只是第一步，运维才决定你能不能长期用。

我后面清理过很多东西：

- 旧 workspace
- 旧 agent 目录
- 旧 session transcript
- 过期的 dashboard 会话
- 不再使用的 Telegram 通道
- 历史配置里的错误命名
- 更新后漂移的字段结构

所以我后来把运维分成固定动作：

### 第一类：状态检查

```bash
openclaw status
openclaw doctor
openclaw config validate
openclaw gateway status
```

### 第二类：服务管理

```bash
openclaw gateway restart
openclaw gateway stop
openclaw gateway status
```

### 第三类：升级维护

```bash
openclaw update
openclaw doctor --fix
```

### 第四类：插件检查

```bash
openclaw plugins list
```

如果你把这些命令当作日常工具，而不是“出问题才想起来用”，OpenClaw 会稳定很多。

---

## 第 10 章：当配置和 prompt 都不够用了，就要开始读源码

这是从中级用户迈向高级用户的分界线。

我这次真正进入源码修改，是因为共享群里出现了一个很具体的问题：

- 人类明明 `@ceo`
- 但 `trade` 还是会抢答
- 看起来像模型理解错了
- 实际上根因在飞书插件的入站解析层

更准确地说，早期逻辑里，bot 往往只知道：

- 这条消息“有没有 mention”
- 但不知道“mention 到底指向了谁”

于是 `trade` 只看到“群里有人说了一句话”，却看不到“这句话其实是在 `@ceo`”。

这类问题靠 prompt 是修不好的。

因为信息在进入模型之前就已经丢了。

---

## 第 11 章：我是怎么确认“该改插件源码，而不是继续调 prompt”的

排查这类问题，我后来遵守一个顺序：

1. 看用户在群里的原始行为
2. 看日志里收到的入站事件
3. 看 session transcript 里 agent 实际看到了什么
4. 判断问题发生在：
   - 配置层
   - 提示词层
   - 渠道适配层

这次结论非常清楚：

- 飞书群里人类 `@ceo`
- `trade` 仍收到消息
- 且它实际看到的文本里，已经没有 `@ceo` 这层信息了

所以问题不在模型，而在插件。

这个判断很重要。

因为一旦根因在插件层，继续调 prompt 只会越调越乱。

---

## 第 12 章：为什么改安装目录下的源码会生效

很多人会以为：

“我是通过 Homebrew 装的 OpenClaw，改源码不可能生效吧？”

其实这要分开看：

- Homebrew 管的是命令入口和 Node 环境
- OpenClaw 主包仍然在全局 Node 目录里
- Feishu 插件也是这个包的一部分
- 而且这版插件是运行时加载的

我这边确认到的链路是：

- 命令入口：`/opt/homebrew/bin/openclaw`
- 包目录：`/opt/homebrew/lib/node_modules/openclaw`
- Feishu 插件：`/opt/homebrew/lib/node_modules/openclaw/extensions/feishu`
- 插件源码目录：`/opt/homebrew/lib/node_modules/openclaw/extensions/feishu/src`

所以只要改的是这里：

```text
/opt/homebrew/lib/node_modules/openclaw/extensions/feishu/src/
```

再重启：

```bash
openclaw gateway restart
```

改动就会生效。

也就是说，**你是通过 Homebrew 启动 OpenClaw，但插件本体是从全局安装目录里运行时加载的。**

---

## 第 13 章：源码修改的正确姿势，不是“大胆改”，而是“证据驱动地小改”

我这次改插件，不是重构整个飞书模块，而是做了三个很具体的修复：

1. 保留显式 mention 目标
2. 只移除“当前 bot 自己”的 mention，不再把别人一起删掉
3. 如果消息明确 `@` 了别的已配置 bot，而当前 bot 没被点名，就直接跳过

这类修法的原则是：

- 改最小闭环
- 只修你已经拿到证据的问题
- 改完立刻回归测试
- 不要把渠道适配层的 bug 甩给模型

这一步之后，我实际验证到的效果就是：

- `@ceo` 时，`trade` 不再抢答
- `@trade` 时，`ceo` 不再抢答
- 同时 `@ceo @trade`，双方都能进入判断

这说明修的是路由逻辑，而不是“把模型调乖了”。

这里还要补一个很重要的边界说明：

**这次源码修复，不等于“群里只要没 `@` 自己，机器人就永远不回复”。**

真实逻辑是这样的：

- 如果一条群消息**明确 `@` 了某个已配置 bot**
- 而当前 bot **不在被点名名单里**
- 那当前 bot 才会被硬跳过

也就是说，这次修复拦的是“定向点名却被其他 bot 抢答”这种情况，而不是把所有未点名消息全部封死。

把它展开成行为矩阵会更清楚：

- 人类 `@ceo 你来处理`
  - `ceo` 可以回复
  - `trade`、`writer`、`crypto` 应该跳过

- 人类 `@ceo @trade 你们一起看`
  - `ceo` 和 `trade` 都可以进入判断
  - 没被点到的 bot 跳过

- 人类在群里发了一条普通消息，没有 `@` 任何 bot
  - 这次源码修复不会直接拦掉这条消息
  - 如果当前群配置允许常驻观察，那么机器人仍然可以看到这条消息
  - 最终是否回复，还是由群规则和 system prompt 决定

- 人类 `@` 的是普通成员，而不是某个已配置 bot
  - 也不会触发这条“非目标 bot 硬跳过”逻辑

所以更准确的说法是：

**我改的是“尊重明确点名”，不是“强制只有被 `@` 才能回复”。**

这也是为什么源码修复之后，系统仍然保留了“群内常驻观察 + 有价值才回复”的能力，只是不会再在用户已经明确点名别的 bot 时乱插嘴。

---

## 第 14 章：高级用户必须接受的一件事，升级会覆盖你的补丁

如果你走到了源码修改这一步，就一定要接受一个现实：

**`openclaw update` 很可能覆盖你在安装目录里的改动。**

所以高级用户最好养成两个习惯：

1. 快修时可以直接 patch 安装目录
2. 长期要保留补丁记录，或者做本地覆盖插件

否则下一次更新后，之前修好的问题可能又回来了。

这一点对 OpenClaw 很重要，因为它本身迭代很快，渠道插件也会跟着变化。

---

## 第 15 章：我对 OpenClaw 的最终评价

如果你的目标只是“接一个 AI 机器人聊天”，OpenClaw 未必是成本最低的选择。

但如果你要的是下面这些能力，它非常值得折腾：

- 多 agent 分工
- workspace 记忆
- 飞书接入
- 插件和技能机制
- 心跳、调度、内部 handoff
- 本地可控
- 出问题时能自己修

它更像一套你自己的 AI 基础设施，而不是一款现成 App。

你会经历三个阶段：

- 小白阶段：先装起来
- 中级阶段：把配置和目录收口
- 高级阶段：真正看懂 OpenClaw 是怎么工作的

到了第三阶段，你才算真正把它掌控住。

---

## 第 16 章：给后来者的 10 条建议

1. 先跑单 agent，再扩展多 agent。
2. 命名一开始就统一，agent、workspace、accountId 尽量一致。
3. `openclaw.json` 要成为唯一真源。
4. 不用的通道及时删，不要留历史垃圾。
5. dashboard 出现怪现象时，先查状态和索引，不要先怪模型。
6. 共享群一定要配 group 级规则。
7. 多 bot 协作优先走内部 handoff，不要只靠群里互相 `@`。
8. 每次更新后跑 `openclaw doctor --fix`。
9. 真遇到底层问题时，先查日志和 transcript，再考虑改源码。
10. 改过安装包源码，就要默认下次升级可能覆盖补丁。

---

## 结尾：这套系统真正难的，不是安装，而是持续维护

我这次完整走下来，最大的感受是：

装上 OpenClaw 并不难。

难的是让它在你的机器上长期可用、角色不乱、群聊不炸、会话清爽、升级不翻车。

真正把它用好，靠的不是某个神奇模型，也不是某条万能 prompt。

而是你有没有把下面这些都收口：

- 配置
- 命名
- 目录
- 会话
- 渠道
- 插件
- 升级策略
- 排障路径

当这些都收住之后，OpenClaw 才真正从“能跑”变成“能用”。

---

## 这个系列怎么拆开发最合适

如果你想把它做成一个系列，我建议拆成 9 篇：

### 第 1 篇：OpenClaw 是什么，适合什么人

讲定位、能力边界、为什么它不是普通聊天机器人。

### 第 2 篇：从 0 安装 OpenClaw，并跑通第一个 agent

讲安装、目录、最小配置、基本验证。

### 第 3 篇：把 OpenClaw 接上飞书

讲飞书权限、事件订阅、网关、基础排障。

### 第 4 篇：Skills 才是 OpenClaw 真正的能力放大器

讲为什么没 skills 的 OpenClaw 很笨、常见 skills 怎么选、怎么装、怎么验证 ready。

### 第 5 篇：从单 agent 扩展到四个机器人团队

讲 `agents.list`、`bindings`、`accounts`、workspace 结构。

### 第 6 篇：多机器人共享群的真实坑

讲群策略、`replyToMode`、`replyInThread`、抢答、内部 handoff。

### 第 7 篇：当配置不够用时，如何读日志、排障、改插件源码

讲高级用户视角，源码路径、热修、升级覆盖风险。

### 第 8 篇：市面上的各种 OpenClaw，架构差异、如何选择，以及安全性

讲官方标准形态、App 托管、远程 Gateway、Docker、社区一键部署壳，以及如何根据自己的需求选择，并把安全边界讲清楚。

### 第 9 篇：调教 OpenClaw 过程中的关键问题与技术点复盘

讲这次真实调教过程里出现过的核心问题：配置漂移、历史状态污染、provider 切换、共享群抢答、bot-to-bot 不可靠、热修与升级覆盖等。
