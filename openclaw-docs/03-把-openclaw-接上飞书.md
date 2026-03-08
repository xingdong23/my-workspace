# 第 3 篇：把 OpenClaw 接上飞书

> **🎯 本篇目标**：完成飞书通道接入，实现私聊和群聊消息收发

---

## 📋 前置条件

- [ ] 已完成 [第 2 篇](./02-从0安装-openclaw-并跑通第一个-agent.md)，Dashboard 能和 AI 对话
- [ ] 有飞书管理员权限（能创建企业自建应用）
- [ ] OpenClaw 所在设备能正常访问飞书开放平台（长连接模式不需要公网回调地址）

---

## 1. 整体流程（6 步）

```
1. 创建飞书应用      → 拿到 App ID 和 App Secret
2. 开启机器人能力    → 让应用能被私聊
3. 配置权限并发布    → 让机器人能收发消息
4. 配置 openclaw.json → 写入 App ID 和 App Secret
5. 配置事件订阅      → 切换长连接并保存
6. 测试验证         → 私聊 + 群聊都通
```

---

## 2. 步骤一：创建飞书应用

访问 https://open.feishu.cn/ → 点击右上角「开发者后台」→「创建企业自建应用」

创建后，在「凭证与基础信息」页面复制：

| 字段 | 示例 | 用途 |
|------|------|------|
| **App ID** | `cli_a1b2c3d4e5f6g7h8` | 填到 openclaw.json |
| **App Secret** | `aB1cD2eF3gH4iJ5kL6mN7oP8` | 填到 openclaw.json |

---

## 3. 步骤二：开启机器人能力（私聊必须！）

**这一步不做，机器人无法被私聊。**

左侧菜单 → 「应用功能」→「机器人」→ 打开「启用机器人」开关

确保勾选：
- ✅ 在群聊中可用
- ✅ 在私聊中可用

---

## 4. 步骤三：配置权限并发布

### 4.1 添加权限

左侧菜单 → 「权限管理」→ 搜索并添加：

| 权限名称 | 搜索关键词 |
|---------|-----------|
| 获取与发送单聊、群聊消息 | `im:message` |
| 以应用身份读取群聊消息 | `im:message:receive_as_bot` |
| 获取用户基本信息 | `contact:user.base` |
| 获取群组信息 | `im:chat` |

### 4.2 发布版本（必须！）

**权限改了必须发布才生效。**

左侧菜单 → 「版本管理与发布」→「创建版本」→「保存并发布」

---

## 5. 步骤四：配置 openclaw.json

打开 `~/.openclaw/openclaw.json`，添加飞书配置：

```json
{
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
          "name": "OpenClaw CEO",
          "appId": "cli_你的AppID",
          "appSecret": "你的AppSecret"
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
    }
  ]
}
```

### 5.1 关键字段

| 字段 | 值 | 说明 |
|------|-----|------|
| `dmPolicy` | `"open"` | 允许私聊 |
| `groupPolicy` | `"open"` | 允许群聊 |
| `requireMention` | `false` | 不需要 @ 也能回复 |
| `streaming` | `false` | 关闭流式，更稳定 |

### 5.2 重启生效

```bash
openclaw config validate
openclaw gateway restart
openclaw status
```

**注意：** 只有这一步里的 `appId` 和 `appSecret` 已经配置正确，并且 `openclaw gateway restart` 后生效，后面的飞书长连接设置才能正常保存并生效。

---

## 6. 步骤五：配置事件订阅（长连接）

左侧菜单 → 「事件与回调」→ 「事件配置」

1. 在「订阅方式」里选择「使用长连接接收事件」
2. 点击「编辑」
3. 这里**不需要填写请求网址、加密策略或其他回调信息**，直接点击「保存」
4. 在下方确认已添加事件 `im.message.receive_v1`；如果没有，就手动补上

---

## 7. 测试私聊

1. 打开飞书，搜索你的机器人名称
2. 发消息：`你好`
3. 机器人应该回复

**如果没反应，检查：**
- [ ] 机器人能力是否开启？
- [ ] 「在私聊中可用」是否勾选？
- [ ] 版本是否已发布？

---

## 8. 测试群聊

1. 创建测试群 → 群设置 → 添加机器人 → 选择你的应用
2. 发消息：`你好`
3. 机器人应该在群里回复

**如果没反应，检查：**
- [ ] 权限 `im:message:receive_as_bot` 是否添加？
- [ ] 事件 `im.message.receive_v1` 是否添加？
- [ ] 版本是否已发布？

---

## ⚠️ 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 私聊找不到机器人 | 机器人能力未开启 | 应用功能 → 机器人 → 开启 |
| 私聊/群聊没反应 | 版本未发布 | 版本管理与发布 → 发布 |
| 长连接保存后不生效 | `appId` / `appSecret` 未配置或未生效 | 检查 `openclaw.json` 后执行 `openclaw gateway restart` |
| 配置报错 | JSON 格式错误 | 检查逗号、引号、括号 |

---

## ✅ 验证清单

- [ ] 飞书应用已创建，拿到 App ID 和 App Secret
- [ ] **机器人能力已开启**（私聊必须）
- [ ] 权限已配置：`im:message`、`im:message:receive_as_bot`
- [ ] openclaw.json 已配置 channels 和 bindings
- [ ] `openclaw config validate` 通过
- [ ] `openclaw gateway restart` 成功
- [ ] **版本已发布**
- [ ] 事件订阅已切换为长连接，且已添加 `im.message.receive_v1`
- [ ] 私聊测试通过
- [ ] 群聊测试通过

---

## 🔗 下一步

飞书接通后，下一步让 OpenClaw 变强：

**→ [第 4 篇：Skills 才是 OpenClaw 真正的能力放大器](./04-skills-才是-openclaw-真正的能力放大器.md)**
