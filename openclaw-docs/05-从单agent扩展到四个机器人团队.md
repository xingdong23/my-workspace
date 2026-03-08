# 第 5 篇：从单 agent 扩展到四个机器人团队

> **🎯 本篇目标**：理解多 agent 架构，正确配置团队角色和协作关系

很多人把"新增一个 agent"理解成"复制一个目录，换个名字"。

真实情况是：
> 新增 agent 其实是在新增一整套运行单元。

---

## 📋 前置条件

- [ ] 已完成 [第 4 篇](./04-skills-才是-openclaw-真正的能力放大器.md)
- [ ] 单 agent 已经稳定运行
- [ ] 已规划好多角色分工

---

## 1. 多 Agent 架构概览

### 1.1 单 Agent vs 多 Agent

```
┌─────────────────────────────────────────────────────────────────┐
│                   单 Agent 架构                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   飞书消息 ──▶ CEO Agent ──▶ 回复                               │
│                                                                 │
│   简单，但能力单一                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   多 Agent 架构                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   飞书消息 ──▶ 路由判断 ──┬──▶ CEO Agent   (协调、分析)         │
│                          ├──▶ Trade Agent  (技术、执行)         │
│                          ├──▶ Writer Agent (内容、写作)         │
│                          └──▶ Crypto Agent (研究、辅助)         │
│                                                                 │
│   复杂，但分工明确、能力专精                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 多 Agent 的核心价值

| 价值 | 说明 |
|------|------|
| **专业分工** | 每个 agent 只做自己擅长的事 |
| **上下文隔离** | 各自的记忆不互相干扰 |
| **角色稳定** | 不会"什么都能做 = 什么都做不好" |
| **可扩展** | 需要新能力时加新 agent，不改旧 agent |

---

## 2. 团队角色设计

### 2.1 我的四个角色

| Agent | 职责 | Skills | 性格特点 |
|-------|------|--------|----------|
| **ceo** | 产品、运营、分析、协调 | feishu-doc, agent-reach | 果断、全局观 |
| **trade** | 技术、交易、系统执行 | github, coding-agent | 严谨、执行导向 |
| **writer** | 写文章、文稿整理、内容输出 | copywriting, feishu-doc | 细致、表达力强 |
| **crypto** | 加密货币研究和辅助判断 | agent-reach, content-strategy | 数据驱动、谨慎 |

### 2.2 角色边界原则

```
┌─────────────────────────────────────────────────────────────────┐
│                  角色边界设计原则                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ✅ DO:                                                        │
│   • 每个角色有明确的职责范围                                    │
│   • 技能配置与职责匹配                                          │
│   • 身份文档清晰定义边界                                        │
│                                                                 │
│   ❌ DON'T:                                                     │
│   • 所有 agent 都配同样的 skills                               │
│   • 角色职责重叠模糊                                            │
│   • 让一个 agent 做所有事                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. 每个 Agent 需要的完整配置

### 3.1 配置清单

新增一个 agent，你需要同时配置：

| 配置项 | 位置 | 说明 |
|--------|------|------|
| `agents.list[]` | openclaw.json | Agent 定义 |
| `agentDir` | 文件系统 | 运行目录 |
| `workspace/` | 文件系统 | 身份和记忆 |
| `channels.feishu.accounts` | openclaw.json | 飞书账号 |
| `bindings[]` | openclaw.json | 路由绑定 |
| `heartbeat` | workspace/ | 定时任务（可选） |

### 3.2 目录结构

```
~/.openclaw/
├── openclaw.json                    # 主配置
│
├── agents/                          # 运行目录
│   ├── ceo/agent/
│   ├── trade/agent/
│   ├── writer/agent/
│   └── crypto/agent/
│
├── workspace-ceo/                   # CEO 身份和记忆
│   ├── IDENTITY.md
│   ├── AGENTS.md
│   ├── MEMORY.md
│   └── HEARTBEAT.md
│
├── workspace-trade/                 # Trade 身份和记忆
│   ├── IDENTITY.md
│   ├── MEMORY.md
│   └── ...
│
├── workspace-writer/                # Writer 身份和记忆
│   └── ...
│
└── workspace-crypto/                # Crypto 身份和记忆
    └── ...
```

---

## 4. 完整配置示例

### 4.1 openclaw.json 完整配置

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
        "workspace": "/Users/<USER>/.openclaw/workspace-ceo",
        "agentDir": "/Users/<USER>/.openclaw/agents/ceo/agent",
        "identity": {
          "name": "CEO",
          "theme": "product, operations, and analysis",
          "emoji": "💼"
        }
      },
      {
        "id": "trade",
        "workspace": "/Users/<USER>/.openclaw/workspace-trade",
        "agentDir": "/Users/<USER>/.openclaw/agents/trade/agent",
        "identity": {
          "name": "Trade",
          "theme": "technical execution and trading",
          "emoji": "📊"
        }
      },
      {
        "id": "writer",
        "workspace": "/Users/<USER>/.openclaw/workspace-writer",
        "agentDir": "/Users/<USER>/.openclaw/agents/writer/agent",
        "identity": {
          "name": "Writer",
          "theme": "content creation and editing",
          "emoji": "✍️"
        }
      },
      {
        "id": "crypto",
        "workspace": "/Users/<USER>/.openclaw/workspace-crypto",
        "agentDir": "/Users/<USER>/.openclaw/agents/crypto/agent",
        "identity": {
          "name": "Crypto",
          "theme": "cryptocurrency research and analysis",
          "emoji": "₿"
        }
      }
    ]
  },
  "channels": {
    "feishu": {
      "enabled": true,
      "defaultAccount": "ceo",
      "dmPolicy": "open",
      "groupPolicy": "open",
      "requireMention": false,
      "streaming": false,
      "blockStreaming": true,
      "accounts": {
        "ceo": {
          "enabled": true,
          "name": "Feishu CEO",
          "appId": "<CEO_APP_ID>",
          "appSecret": "<CEO_APP_SECRET>"
        },
        "trade": {
          "enabled": true,
          "name": "Feishu Trade",
          "appId": "<TRADE_APP_ID>",
          "appSecret": "<TRADE_APP_SECRET>"
        },
        "writer": {
          "enabled": true,
          "name": "Feishu Writer",
          "appId": "<WRITER_APP_ID>",
          "appSecret": "<WRITER_APP_SECRET>"
        },
        "crypto": {
          "enabled": true,
          "name": "Feishu Crypto",
          "appId": "<CRYPTO_APP_ID>",
          "appSecret": "<CRYPTO_APP_SECRET>"
        }
      }
    }
  },
  "bindings": [
    {
      "agentId": "ceo",
      "match": {
        "channel": "feishu",
        "accountId": "ceo"
      }
    },
    {
      "agentId": "trade",
      "match": {
        "channel": "feishu",
        "accountId": "trade"
      }
    },
    {
      "agentId": "writer",
      "match": {
        "channel": "feishu",
        "accountId": "writer"
      }
    },
    {
      "agentId": "crypto",
      "match": {
        "channel": "feishu",
        "accountId": "crypto"
      }
    }
  ],
  "tools": {
    "agentToAgent": {
      "allow": true
    }
  }
}
```

### 4.2 配置字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `agents.list[].id` | ✅ | Agent 唯一标识 |
| `agents.list[].default` | 可选 | 是否为默认 agent |
| `agents.list[].workspace` | ✅ | Workspace 目录路径 |
| `agents.list[].agentDir` | ✅ | 运行目录路径 |
| `channels.feishu.accounts` | ✅ | 飞书账号配置 |
| `bindings` | ✅ | Agent 与 Channel 的绑定关系 |
| `tools.agentToAgent.allow` | 推荐 | 是否允许 agent 间协作 |

---

## 5. 创建 Workspace 文件

### 5.1 IDENTITY.md 模板

以 `writer` 为例，在 `~/.openclaw/workspace-writer/IDENTITY.md`：

```markdown
# Writer Agent 身份定义

## 我是谁

我是团队的内容创作专家，负责：
- 文章撰写和编辑
- 文稿整理和润色
- 内容结构优化
- 文档格式规范

## 我的性格

- 表达清晰、逻辑性强
- 注重细节和准确性
- 善于组织和结构化信息

## 我的职责边界

### ✅ 我负责的
- 撰写各类文章和文档
- 编辑和润色现有内容
- 优化内容结构和可读性
- 格式化和排版

### ❌ 我不负责的
- 技术决策和代码实现（找 Trade）
- 产品策略和运营分析（找 CEO）
- 加密货币深度研究（找 Crypto）

## 我的协作方式

- 接到写作任务后，先确认主题和受众
- 完成初稿后会主动请求反馈
- 需要技术内容时，会请 Trade 协助
```

### 5.2 AGENTS.md（团队认知）

在 `workspace-ceo/AGENTS.md`：

```markdown
# 团队成员

## CEO (我)
- 职责：产品决策、运营分析、团队协调
- 擅长：全局视角、决策分析

## Trade
- 职责：技术实现、系统执行、交易操作
- 擅长：代码、技术问题

## Writer
- 职责：内容创作、文档整理
- 擅长：写作、表达

## Crypto
- 职责：加密货币研究、市场分析
- 擅长：数据分析、趋势判断

## 协作规则

1. 遇到技术问题 → 找 Trade
2. 需要写文章 → 找 Writer
3. 需要市场分析 → 找 Crypto
4. 需要协调决策 → 找 CEO
```

---

## 6. Agent 间协作配置

### 6.1 启用 agentToAgent

```json
{
  "tools": {
    "agentToAgent": {
      "allow": true
    }
  }
}
```

### 6.2 协作方式

```
┌─────────────────────────────────────────────────────────────────┐
│                  Agent 间协作方式                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   方式 1: 内部 Handoff（推荐）                                  │
│   ───────────────────────────                                   │
│   用户 → CEO → (内部调用) → Trade → CEO → 用户                  │
│   用户只看到 CEO 在回复                                         │
│                                                                 │
│   方式 2: 群里 @（不推荐）                                      │
│   ───────────────────────────                                   │
│   用户 → CEO → @trade → Trade 回复                              │
│   问题：飞书事件传递不稳定                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. 验证配置

### 7.1 验证命令

```bash
# 1. 验证配置格式
openclaw config validate

# 2. 检查所有 agent 状态
openclaw status

# 3. 查看特定 agent
openclaw agent show ceo
openclaw agent show trade
```

### 7.2 预期输出

**`openclaw status` 应显示：**
```
Agents:
  ceo (default)  ✓ running
  trade          ✓ running
  writer         ✓ running
  crypto         ✓ running

Channels:
  feishu/ceo     ✓ connected
  feishu/trade   ✓ connected
  feishu/writer  ✓ connected
  feishu/crypto  ✓ connected

Gateway: running on port 3000
```

---

## ⚠️ 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 新 agent 不响应 | binding 未配置 | 检查 bindings 列表 |
| 所有 agent 都回复同一消息 | 路由配置错误 | 检查 accountId 对齐 |
| agent 间无法协作 | agentToAgent 未启用 | 设置 `tools.agentToAgent.allow: true` |
| workspace 报错 | 目录不存在 | 创建 workspace 目录和文件 |
| 飞书账号冲突 | 多 agent 用同一 App | 每个 agent 用独立飞书应用 |

---

## 8. 扩展顺序建议

```
┌─────────────────────────────────────────────────────────────────┐
│                  推荐扩展顺序                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   第 1 步：单 agent 跑稳                                        │
│   └── 确认 ceo 单独工作正常                                     │
│                                                                 │
│   第 2 步：加入第二个 agent                                     │
│   └── 加入 trade，确认 binding 正确                             │
│                                                                 │
│   第 3 步：测试内部协作                                         │
│   └── 确认 agentToAgent 可用                                    │
│                                                                 │
│   第 4 步：加入更多 agent                                       │
│   └── 按 need 逐步添加 writer、crypto                           │
│                                                                 │
│   第 5 步：共享群策略                                           │
│   └── 把多个 bot 放入同一群，配置协作规则                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅ 本篇验证清单

- [ ] 已规划好角色分工
- [ ] `openclaw.json` 中 agents.list 配置完整
- [ ] 每个 agent 都有独立的 workspace 目录
- [ ] 每个 agent 都有对应的飞书账号配置
- [ ] bindings 配置正确，agentId 与 accountId 对齐
- [ ] `openclaw config validate` 通过
- [ ] `openclaw status` 显示所有 agent 运行中
- [ ] agentToAgent 已启用

---

## 🔗 下一步

当多个 agent 都配置好后，下一步是解决共享群的协作问题：

**→ [第 6 篇：多机器人共享群的真实坑](./06-多机器人共享群的真实坑.md)**
