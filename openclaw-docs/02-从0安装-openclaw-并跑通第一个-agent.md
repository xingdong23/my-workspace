# 第 2 篇：从 0 安装 OpenClaw，并跑通第一个 Agent

> **🎯 本篇目标**：安装完成 → 打开 Dashboard → 能和 AI 对话

**跑通的标准只有一个：在 Dashboard 里能和 AI 对话。**

---

## 官方资源

| 资源 | 地址 |
|------|------|
| **GitHub** | https://github.com/openclaw/openclaw |
| **官方文档** | https://docs.openclaw.ai |
| **最新版本** | 查看 [Releases](https://github.com/openclaw/openclaw/tags) |

> 💡 当前最新版本发布于 2026.3.2，建议安装最新版以获得最佳体验。

---

## 1. 安装

```bash
# 安装 Node.js（如果没有）
brew install node

# 安装 OpenClaw
npm install -g openclaw

# 验证
openclaw --version
```

---

## 2. 启动

```bash
# 初始化（首次运行会自动创建配置）
openclaw doctor

# 启动网关
openclaw gateway start

# 打开 Dashboard
open http://localhost:3000
```

**如果 Dashboard 打开了，你已经完成 80%！**

---

## 3. 配置 API Key（必须做）

安装后虽然会自动生成配置，但 **API Key 需要你自己填**。

### 3.1 先获取 API Key

你需要选择一个模型提供商：

| Provider | 获取地址 | 说明 |
|----------|---------|------|
| **智谱 GLM** | https://open.bigmodel.cn | 国内可用，推荐新手 |
| **OpenAI** | https://platform.openai.com | 需要 VPN |
| **Claude** | https://console.anthropic.com | 需要 VPN |
| **DeepSeek** | https://platform.deepseek.com | 国内可用 |

> 💡 **新手建议**：如果没有海外账号，先用智谱或 DeepSeek，注册就能拿到 Key。

### 3.2 找到配置文件

```bash
# 打开配置文件
code ~/.openclaw/openclaw.json

# 或者用 vim
vim ~/.openclaw/openclaw.json
```

### 3.3 填入 API Key

打开配置文件后，找到 `models.providers` 下你使用的提供商，把 `apiKey` 字段的值改成你的 Key：

**智谱 GLM 示例：**
```json
{
  "models": {
    "providers": {
      "zai": {
        "baseUrl": "https://open.bigmodel.cn/api/coding/paas/v4",
        "apiKey": "这里填你的智谱 API Key",
        "api": "openai-completions",
        "models": [...]
      }
    }
  }
}
```

**OpenAI 示例：**
```json
{
  "models": {
    "providers": {
      "openai": {
        "baseUrl": "https://api.openai.com/v1",
        "apiKey": "这里填你的 OpenAI API Key（sk-xxx）",
        "api": "openai-completions",
        "models": [...]
      }
    }
  }
}
```

**DeepSeek 示例：**
```json
{
  "models": {
    "providers": {
      "deepseek": {
        "baseUrl": "https://api.deepseek.com/v1",
        "apiKey": "这里填你的 DeepSeek API Key",
        "api": "openai-completions",
        "models": [...]
      }
    }
  }
}
```

### 3.4 保存并重启

```bash
# 保存配置文件后，重启网关
openclaw gateway restart

# 验证配置是否生效
openclaw status
```

### 3.5 常见问题

| 问题 | 解决 |
|------|------|
| 找不到 `apiKey` 字段 | 搜索 `"apiKey"` 或 `"providers"` |
| 配置改了没生效 | 确保 JSON 格式正确（逗号、引号），然后 `gateway restart` |
| 不确定用哪个 Provider | 看配置文件里已有的 `providers` 下面有哪些，选一个 |
| Key 填了但还是报错 | 检查 Key 是否复制完整，没有多余空格 |

---

## 4. 验证跑通

1. 打开 Dashboard：`http://localhost:3000`
2. 找到聊天界面
3. 发送：`你好`
4. **收到回复 = 跑通了！**

---

## 常见问题

| 问题 | 解决 |
|------|------|
| Dashboard 打不开 | `openclaw gateway status` 确认网关在运行 |
| 发消息没反应 | 检查 API Key 是否正确 |
| 配置改了不生效 | `openclaw gateway restart` |
| 3000 端口被占用 | `lsof -i :3000` 查看并杀掉占用进程 |

---

## 排障命令

```bash
openclaw status           # 查看状态
openclaw doctor           # 检查环境
openclaw logs --lines 50  # 看日志
```

---

## ✅ 验证清单

**唯一标准：在 Dashboard 里能和 AI 对话。**

---

## 下一步

**→ [第 3 篇：把 OpenClaw 接上飞书](./03-把-openclaw-接上飞书.md)**
