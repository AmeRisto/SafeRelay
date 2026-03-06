# SafeRelay - 部署指南

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/qianqi32/SafeRelay)

> 💡 **推荐使用一键部署**，更新时只需点击 GitHub 仓库的 `Sync fork` 即可自动同步。

---

## 部署方式选择

| 方式 | 难度 | 适合人群 | 特点 |
|------|------|----------|------|
| [一键部署](#一键部署推荐) | ⭐ 简单 | 所有用户 | 自动创建 KV，后续可自动更新 |
| [手动部署](#手动部署详细步骤) | ⭐⭐ 中等 | 不想用 GitHub 的用户 | 完全手动控制每一步 |

---

## 前置准备

无论选择哪种部署方式，都需要先准备：

1. **Telegram Bot Token**：在 TG 上找 [@BotFather](https://t.me/BotFather) 创建机器人获取
2. **Admin UID**：在 TG 上找 [@userinfobot](https://t.me/userinfobot) 获取你自己的 User ID
3. **Webhook 密钥**：访问 [UUID 生成器](https://www.uuidgenerator.net/) 生成一个随机 UUID

---

## 步骤 1：配置 Cloudflare Turnstile（必需）

前往 `Cloudflare Dashboard` → `应用程序安全` → `Turnstile`：

1. 点击 **添加小组件** 按钮
2. **小组件名称**：填写 `saferelay`（或其他名称）
3. **主机名管理**：点击 **添加主机名** 按钮
   - 选择 **添加自定义主机名**
   - 填写你的 Workers 域名，例如 `yourname.workers.dev`
   - 点击输入框旁边的 **添加** 按钮
   - 点击下方的 **添加** 按钮确认
4. 点击 **创建** 按钮
5. **保存密钥**：创建成功后会显示
   - **站点密钥 (Site Key)**：复制保存
   - **密钥 (Secret Key)**：复制保存

> ⚠️ **重要**：这两个密钥稍后要填写到代码中！

---

## 一键部署（推荐）

### 2. Fork 本仓库

点击右上角的 `Fork` 按钮，将本仓库 Fork 到您的 GitHub 账户。

> **注意**：取消勾选 "Copy the main branch only"

### 3. 配置 Turnstile 密钥

在你的 Fork 仓库中，编辑 `worker.js`（第 9-10 行）：

```javascript
const CF_TURNSTILE_SITE_KEY = '0x4AAAAAAAXXXXXXXXXXXXXXXXXXXX';   // 替换为你的 Site Key
const CF_TURNSTILE_SECRET_KEY = '0x4AAAAAAAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX';  // 替换为你的 Secret Key
```

### 4. 一键部署

点击上方 **Deploy to Cloudflare Workers** 按钮：

1. 登录 Cloudflare 账户
2. 修改项目名称为 `SafeRelay2`（或其他名称）
3. 点击 `Deploy`

> 部署时会自动创建 KV 命名空间，无需手动绑定！

### 5. 连接自己的 Fork（用于后续更新）

1. 部署完成后，在同一页面打开 Worker 的 `设置`
2. 找到 `Git 存储库` 部分，点击 `断开连接`
3. 重新连接到你 Fork 的仓库（`qianqi32/SafeRelay`）
4. 选择 `main` 分支

> 现在你可以删除 `SafeRelay2` 仓库了

### 6. 设置环境变量

在 Worker 的 `设置` → `变量` 中添加：

| 变量名 | 类型 | 说明 | 示例 |
|:------:|:----:|:-----|:----:|
| `ENV_BOT_TOKEN` | Secret | Bot Token | `123456:ABC-DEF...` |
| `ENV_BOT_SECRET` | Secret | Webhook 密钥 | `random_string_123` |
| `ENV_ADMIN_UID` | 明文 | 管理员 User ID | `123456789` |

点击 `保存并部署`

### 7. 激活 Webhook

浏览器访问：
```
https://saferelay.yourname.workers.dev/registerWebhook
```

看到 `Ok` 即部署成功！

---

## 手动部署（详细步骤）

如果你不想使用 GitHub，可以直接在 Cloudflare Dashboard 手动部署。

### 步骤 2：创建 KV 命名空间

前往 `Cloudflare Dashboard` → `Storage & Databases` → `Workers KV`：

1. 点击 `Create a Namespace`（创建命名空间）
2. 命名为 `saferelay_kv`（或其他你喜欢的名字）
3. 点击 `Add`（添加）

### 步骤 3：创建 Worker

前往 `Cloudflare Dashboard` → `Compute & AI` → `Workers & Pages`：

1. 点击 `Create Application`（创建应用程序）
2. 选择 `Hello World` 模板
3. `Worker Name` 填写 `saferelay`（或其他你喜欢的名字）
4. 点击 `Deploy`（部署）

### 步骤 4：编辑代码

1. 进入 `Workers & Pages` → 你的 Worker 名称 → `Edit Code`（编辑代码）
2. 将 [worker.js](./worker.js) 的内容完整复制粘贴进去，覆盖原有代码
3. **配置 Turnstile 密钥**：找到第 9-10 行，填入刚才保存的密钥：
   ```javascript
   const CF_TURNSTILE_SITE_KEY = '0x4AAAAAAAXXXXXXXXXXXXXXXXXXXX';  // 替换为你的 Site Key
   const CF_TURNSTILE_SECRET_KEY = '0x4AAAAAAAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX';  // 替换为你的 Secret Key
   ```
4. 点击右上角的 `Deploy`（部署）保存

### 步骤 5：绑定 KV

进入 `Workers & Pages` → 你的 Worker 名称 → `Settings`（设置） → `Bindings`（绑定）：

1. 点击 `Add`（添加绑定）
2. 选择 `KV Namespace`（KV 命名空间）
3. `Variable name` **必须填写 `KV`**（必须大写，代码中写死了这个名字）
4. `KV Namespace` 选择刚才创建的 `saferelay_kv`
5. 点击 `Save and deploy`（保存并部署）

### 步骤 6：设置环境变量

进入 `Workers & Pages` → 你的 Worker 名称 → `Settings`（设置） → `Variables`（变量）：

在 `Environment Variables`（环境变量）中添加以下变量：

| 变量名 | 类型 | 说明 | 示例 |
|:------:|:----:|:-----|:----:|
| `ENV_BOT_TOKEN` | Secret | 你的 Bot Token | `123456:ABC-DEF...` |
| `ENV_BOT_SECRET` | Secret | Webhook 密钥（随机字符串） | `random_string_123` |
| `ENV_ADMIN_UID` | Plain text | 管理员的 User ID | `123456789` |

> **注意**：`ENV_BOT_TOKEN` 和 `ENV_BOT_SECRET` 建议设置为 `Secret` 类型以保护安全。

点击 `Save and deploy`（保存并部署）

### 步骤 7：激活 Webhook

部署完成后，在浏览器访问以下 URL 来激活机器人：

```
https://<你的worker域名>/registerWebhook
```

示例：
```
https://saferelay.yourname.workers.dev/registerWebhook
```

如果看到 `Ok`，说明部署成功！

发送 `/start` 给你的机器人，确认可以收到机器人回复。

> 💡 **管理员指令**：详细指令说明请查看 [README.md](./README.md#-管理员指令)

---

## 同步上游（更新机器人）

### 一键部署用户

当本仓库更新时，你可以同步最新代码：

**手动同步：**
1. 打开你的 Fork 仓库
2. 看到顶部提示 "Sync fork" 时，点击它

**自动同步（可选）：**
1. 进入你的 Fork 仓库 → `Actions`
2. 点击 "I understand my workflows, go ahead and enable them"
3. 每天凌晨自动同步上游更新

### 手动部署用户

需要手动复制新版本的 `worker.js` 代码，重新部署：

1. 下载最新版 [worker.js](./worker.js)
2. 进入你的 Worker → `Edit Code`
3. 粘贴新代码（注意保留你的 Turnstile 密钥）
4. 点击 `Deploy`

---

## ⚠️ 注意事项

1. **KV 绑定名称**：请确保 KV Namespace 的变量名绑定为 `KV`，否则机器人无法记忆状态。
2. **KV 延迟**：Cloudflare KV 存在短暂的最终一致性延迟（约 1 分钟）。如果你刚解封用户，可能需要等几十秒才会生效。
3. **Webhook 必须激活**：部署完成后必须访问 `/registerWebhook`，否则机器人无法接收消息。
4. **联合封禁**：使用第三方服务查询，会共享用户 ID，请根据隐私需求决定是否开启。
