# 第 4 篇：Skills 才是 OpenClaw 真正的能力放大器

> **🎯 本篇目标**：理解 skill 是什么、为什么重要、常见来源有哪些、优先补哪些类型

如果前面三篇解决的是“把 OpenClaw 装起来、接起来、跑起来”，那这一篇只讲一件事：

**怎么让 OpenClaw 真的开始有工作能力。**

到这一步，默认你的 OpenClaw 已经安装完成，也已经能正常工作。后面缺什么 skill，原则上都可以直接让 OpenClaw 自己去找、去装、去接入。

所以这一篇不再教你手动研究安装细节，而是先帮你建立一张清晰的“能力地图”。

---

## 📋 前置条件

- [ ] 已完成 [第 3 篇](./03-把-openclaw-接上飞书.md)，OpenClaw 已能正常收发消息
- [ ] `openclaw status` 正常
- [ ] 接受一个前提：从现在开始，缺什么能力，就让 OpenClaw 去补对应的 skill

---

## 1. 先抓住一个原则

**Skill 不是锦上添花，而是 OpenClaw 从“会聊天”变成“会做事”的关键。**

所以从这一篇开始，建议你把精力放在两件事上：

- 先分清自己缺的是哪一类能力
- 再让 OpenClaw 去补对应的 skill

不要再把重点放在“我要不要自己手动装 skill、手动配目录、手动搬文件”这种事情上。

更实用的做法是直接说：

- `帮我找一个能搜索网页的 skill，并安装`
- `帮我装一个能读取 PDF 和 DOCX 的 skill`
- `帮我找一个适合网页操作和截图的 skill`

这一章的价值，不是给你一堆命令，而是让你以后知道：

**缺能力时，应该去哪里找；找到以后，应该优先补什么。**

---

## 2. Skill 到底是什么

你可以把 skill 理解成一个 **能力包**。

它不只是几句 prompt，也不只是一个角色设定。一个真正有用的 skill，通常至少会告诉 OpenClaw：

| 组成部分 | 作用 |
|---------|------|
| 触发条件 | 什么场景下该启用这个 skill |
| 工作步骤 | 遇到这类任务时先做什么、后做什么 |
| 工具建议 | 该用搜索、浏览器、文件处理，还是代码工具 |
| 输出要求 | 最终应该产出什么样的结果 |
| 边界约束 | 哪些事能做，哪些事不要做 |

换句话说：

- plugin / channel 更像“接口”
- skill 更像“方法论 + SOP + 工具使用说明”

这也是为什么 skill 才是 OpenClaw 的能力放大器。

---

## 3. 为什么没有 Skill 的 OpenClaw 其实只是“会聊”

很多人觉得“机器人很笨”，其实不一定是模型不行，而是它没有拿到这类任务的工作路径。

| 任务 | 没有 skill 时 | 有 skill 时 |
|------|---------------|------------|
| 搜索最新信息 | 只能泛泛回答，或者回答过时内容 | 知道去搜索、筛选、汇总 |
| 读取某个网页 | 可能只会说“我无法访问” | 知道怎么抓取、提取、整理 |
| 整理 PDF / 文档 | 只能概念性建议 | 知道怎么提取、转成可处理文本 |
| 操作网页 | 只能描述步骤 | 能打开页面、点击、填写、截图 |
| 处理代码仓库 | 只能给建议 | 知道怎么搜索文件、做规划、整理 PR |

所以 skill 的核心价值，不是“回答更像样”，而是：

- 给 OpenClaw 补上外部信息源
- 给 OpenClaw 补上稳定工作流
- 给 OpenClaw 补上某类任务的专业做法

---

## 4. Skill 和 Plugin / Channel 到底有什么区别

这几个概念很容易混在一起：

| 概念 | 主要解决什么 | 例子 |
|------|-------------|------|
| `channel` | 消息从哪里进来、往哪里出去 | 飞书、Discord、Telegram |
| `plugin` / 集成 | 能不能连上某个服务或能力层 | 模型提供商、消息网关、外部服务 |
| `skill` | 这类任务到底该怎么做 | 搜索、爬取、读文档、做规划、提 PR |

一句话总结：

- channel 解决“消息从哪来”
- plugin 解决“系统能不能接上”
- skill 解决“接上以后会不会干活”

---

## 5. 一般去哪里找 Skill

你现在不需要记住具体安装命令，但要知道常见来源，尤其是**官方源在哪，社区索引在哪**。

通常来说，skill 来源大致分 4 类：

| 来源 | 适合找什么 | 典型例子 |
|------|-----------|---------|
| 官方 registry | 官方公开技能目录，适合直接搜索和发现 | `clawhub.ai`、`openclaw/clawhub` |
| 官方归档仓库 | 看历史版本、看技能原始内容、做备份核查 | `openclaw/skills` |
| 社区整理索引 | 按分类挑 skill，比官方目录更适合“逛” | `VoltAgent/awesome-openclaw-skills`、`clawdbot-ai/awesome-openclaw-skills-zh`、`sundial-org/awesome-openclaw-skills` |
| 专题 / 部分生态仓库 | 某个领域特别深的技能集合 | `BankrBot/skills`、`openclaw/nix-openclaw`、团队本地 skill 库 |

### 5.1 官方仓库

这几个是你这章里应该先知道的官方来源：

| 仓库 / 站点 | 定位 | 什么时候看 |
|------------|------|-----------|
| `https://clawhub.ai/` | 官方 skill 目录站点 | 想直接搜 skill、看详情、交给 OpenClaw 自动安装 |
| `openclaw/clawhub` | ClawHub 站点本身的源码仓库 | 想确认官方 registry 是什么、怎么工作的 |
| `openclaw/skills` | ClawHub 上 skill 的归档仓库 | 想看历史版本、看某个 skill 的原始内容，不作为首选安装入口 |
| `openclaw/nix-openclaw` | 官方 Nix 打包与插件仓库 | 走 Nix 部署，或者想看插件如何把工具和 skill 一起接进 OpenClaw |

这里要特别纠正一个容易漏掉的点：

- `clawhub.ai` 不是一个泛泛的网站链接，它背后对应的是官方仓库 `openclaw/clawhub`
- `openclaw/skills` 也不是社区整理，而是官方公开的 skill 归档
- 真正给读者推荐的入口仍然应该是 `clawhub.ai`，或者直接让 OpenClaw 自己搜索并安装

### 5.2 值得看的社区仓库

除了官方源，社区里现在比较值得看的仓库主要有这些：

| 仓库 | 作用 | 适合谁 |
|------|------|-------|
| `VoltAgent/awesome-openclaw-skills` | 从官方 registry 里做二次筛选和分类，适合按任务类型找 skill | 想快速浏览大量 skill 的人 |
| `clawdbot-ai/awesome-openclaw-skills-zh` | 中文整理版，按场景分类，适合中文读者理解 | 中文用户 |
| `sundial-org/awesome-openclaw-skills` | 挑“热门且常用”的 skill 做清单，适合先从高频技能入手 | 不想一上来面对大而杂目录的人 |
| `BankrBot/skills` | 偏加密、链上、身份和交易等专题技能库 | 只对特定垂类感兴趣的人 |

你可以把它理解成：

- 官方源负责“原始目录和真实来源”
- 社区索引负责“筛选、分类、中文化、导航”
- 专题仓库负责“某一类能力做得更深”

对大多数人来说，先知道 **ClawHub 官方目录 + 1 个社区索引仓库** 就已经够用了。

---

## 6. 最常见的信息源 Skill 都有哪些

如果把 skill 看成“信息源 + 工作流”的组合，那最常见的能力其实就下面几类。

| 信息源 / 任务类型 | 这类 skill 用来干什么 | 常见 skill 例子 |
|------------------|----------------------|----------------|
| 公开网页搜索 | 找最新网页、新闻、资料入口 | `search`、`agent-reach` |
| 指定 URL 内容提取 | 已经知道链接，想直接读正文 | `extract` |
| 整站文档 / 知识库抓取 | 把一个网站或文档站成批拉下来分析 | `crawl`、`research` |
| 真实浏览器页面 | 需要登录、点击、截图、填表单 | `agent-browser`、`playwright`、`playwright-cli` |
| 本地文件 / PDF / Office 文档 | 把文件转成可读文本或 Markdown | `markitdown`、`pdf` |
| 本地代码仓库 | 搜索文件、理解结构、做实现规划 | `file-search`、`ai-assisted-coding`、`project-planner` |
| 工程协作流程 | 整理变更、生成 PR、做交付说明 | `create-pr` |
| 文档与知识整理 | 更新现有文档、按结构维护知识库 | `documentation` |

如果你只记一件事，就记这个：

**大多数 skill，本质上都是在帮 OpenClaw 接上某种信息源，或者给它一套处理该信息源的方法。**

### 6.1 用我自己的 skill 清单举例

如果不想只看抽象分类，你完全可以直接看一套真实的 skill 组合会长什么样。

以我自己这套清单为例，最典型的几条能力线大致是这样的：

| 能力线 | 这层解决什么 | 典型 skill 例子 |
|-------|-------------|----------------|
| 交易 | 交易审计、系统治理、复盘训练 | `trading-pnl-audit`、`trading-system-governor`、`trading-coach` |
| 飞书生态 | 文档、表格、消息、桥接、用户级操作 | `feishu-bitable`、`feishu-bridge`、`feishu-doc-editor`、`feishu-doc-manager`、`feishu-docx-powerwrite`、`feishu-messaging`、`feishu-sheets-skill`、`feishu-user` |
| 数据 / 搜索 | 多平台内容抓取、语义发现、RSS、记忆检索 | `agent-reach`、`qveris-official`、`blogwatcher`、`memory-search`、`gifgrep` |
| 营销 / 内容 | 创意、内容策略、文案打磨、发布规划 | `brainstorming`、`content-strategy`、`copywriting`、`launch-strategy`、`marketing-ideas` |
| 系统 / 安全 | GitHub 操作、浏览器自动化、安全审计、技能市场守卫 | `github`、`browserwing`、`clawsec`、`skill-market-guard` |
| 自我进化 | 反思、自我学习、持续优化 | `self-improving` |

这样看就会很直观：

- 有些 skill 是直接连信息源的
- 有些 skill 是把结果写回飞书的
- 有些 skill 是做交易、营销、发布这种垂直工作流的
- 有些 skill 是在保障你这套系统本身能稳定运行和持续进化

如果再看我本机已经装上的目录，你会发现它还继续往外长：

- 可视化类：`mermaid-skill`、`excalidraw-skill`、`python-dataviz-skill`、`fal-ai`
- 生产力类：`gog`、`summarize`、`weather`、`humanizer`
- 开发类：`frontend-design-ultimate`、`neondb`、`elevenlabs`、`remotion-toolkit`

一套成熟的 OpenClaw skill 清单，通常就应该长成这种“通用能力 + 行业能力 + 系统能力”并存的样子。

### 6.2 一些具体组合例子

真正好用的，不是某一个 skill，而是几类 skill 组合起来。

| 场景 | 可以怎么组合 |
|------|-------------|
| 做交易日报 / 周报 | `trading-pnl-audit` + `trading-system-governor` + `feishu-docx-powerwrite` + `feishu-messaging` |
| 做交易复盘 | `trading-coach` + `feishu-doc-manager` + `python-dataviz` |
| 跟踪外部舆情和信息 | `agent-reach` + `multi-search-engine` + `blogwatcher` + `memory-search` |
| 写活动方案或发布文案 | `brainstorming` + `content-strategy` + `copywriting` + `launch-strategy` + `marketing-ideas` |
| 输出给老板或团队看的结构化图表 | `mermaid-skill` + `excalidraw-skill` + `python-dataviz` |
| 做技能或系统侧安全检查 | `clawsec` + `skill-market-guard` + `github` |
| 维护这套 OpenClaw 自己的稳定性 | `myclaw-guardian` + `myclaw-backup` + `skill-market-guard` |
| 把已有工作流沉淀成 skill | `n8n-to-skill` + `github` + `browserwing` |
| 让 agent 变得越来越像“长期同事” | `self-improving` + `memory-search` + `feishu-doc-manager` |

这类例子比“告诉你有个 search skill、有个 browser skill”更有价值，因为读者会立刻明白：

**skill 不是一个个孤立插件，而是一套可以拼起来干活的能力模块。**

---

## 7. 对大多数人，优先补这 4 类 Skill

不是 skill 越多越好。刚开始最容易装乱，也最容易把能力边界搞混。

更稳的顺序是：

| 优先级 | 先补什么 | 为什么 |
|-------|---------|-------|
| 1 | 搜索 / 研究类 | 先让它能接触外部信息，而不是只靠模型瞎猜 |
| 2 | 浏览器类 | 让它能真正打开页面、操作页面、截取页面 |
| 3 | 文档 / 文件类 | 让它能处理 PDF、DOCX、表格、网页正文 |
| 4 | 代码 / 协作类 | 如果你会拿它干技术活，再补代码规划和 PR 流程 |

换成更直白的话：

- 先补“看世界”的能力
- 再补“动手操作”的能力
- 最后再补“专业工种”的能力

这样最不容易装一堆技能却不知道怎么用。

---

## 8. 这一篇之后你应该怎么用

从这一篇开始，你对 skill 的使用姿势应该变成这样：

1. 先判断自己缺的是哪类能力
2. 再告诉 OpenClaw 去找这类 skill
3. 先装一两个最常用的，不要一次装满
4. 用一段时间，确认真的有帮助，再继续补下一类

例如：

- 你总是要查资料，那就优先补搜索类
- 你总是要读链接，那就补提取和爬取类
- 你总是要操作网页后台，那就补浏览器类
- 你总是要读 PDF、合同、方案，那就补文档处理类

这才是更实用的思路。

不是“先装一堆”，而是“先补短板”。

---

## ✅ 本篇验证清单

- [ ] 知道 skill 是能力包，不只是 prompt
- [ ] 知道 skill 和 channel / plugin 不是一回事
- [ ] 知道从现在开始，skill 可以直接交给 OpenClaw 去找、去装
- [ ] 知道常见 skill 来源有哪几类
- [ ] 能分清自己下一步最该补的是哪一类 skill

---

## 🔗 下一步

当你已经理解 skill 的作用，下一步就不是“继续装能力”，而是开始思考：

**不同角色的 agent 应该怎么分工。**

**→ [第 5 篇：从单 agent 扩展到四个机器人团队](./05-从单agent扩展到四个机器人团队.md)**
