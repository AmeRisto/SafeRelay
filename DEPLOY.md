# SafeRelay - 部署指南

## 一、准备工作

在开始之前，请先准备：

1. **Telegram Bot Token**：在 TG 上找 [@BotFather](https://t.me/BotFather) 创建机器人获取。
2. **Admin UID**：在 TG 上找 [@userinfobot](https://t.me/userinfobot) 获取你自己的 User ID。
3. **生成 Webhook 密钥**：访问 [UUID 生成器](https://www.uuidgenerator.net/) 生成一个随机 UUID 作为 SECRET。

## 二、配置 Cloudflare Turnstile

前往 `Cloudflare Dashboard` -> `应用程序安全` -> `Turnstile`：

1. 点击 **添加小组件** 按钮
2. **小组件名称**：填写 `TG_BOT`（或其他你喜欢的名称）
3. **主机名管理**：点击 **添加主机名** 按钮
   - 选择 **添加自定义主机名**
   - 填写你的 Workers 域名，例如 `user.workers.dev`
   - 点击输入框旁边的 **添加** 按钮
   - 点击下方的 **添加** 按钮确认
4. 点击 **创建** 按钮
5. **保存密钥**：创建成功后会显示
   - **站点密钥 (Site Key)**：复制保存
   - **密钥 (Secret Key)**：复制保存

> ⚠️ **重要**：这两个密钥稍后要填写到代码中！

## 三、创建 KV

前往 `Cloudflare Dashboard` -> `Storage & Databases`（存储和数据库） -> `Workers KV`：

1. 点击 `Create a Namespace`（创建命名空间）
2. 命名为 `TG_BOT_KV`（或其他你喜欢的名字）
3. 点击 `Add`（添加）

## 四、创建 Worker 并配置代码

### 4.1 创建 Worker

前往 `Cloudflare Dashboard` -> `Compute & AI`（计算和 AI） -> `Workers & Pages`：

1. 点击 `Create Application`（创建应用程序）
2. 选择 `Hello World` 模板
3. `Worker Name` 填写 `TG_BOT`（或其他你喜欢的名字）
4. 点击 `Deploy`（部署）

### 4.2 编辑代码

1. 进入 `Workers & Pages` -> 你的 Worker 名称 -> `Edit Code`（编辑代码）
2. 将 [worker.js](./worker.js) 的内容完整复制粘贴进去
3. **配置 Turnstile 密钥**：找到第 9-10 行，填入刚才保存的密钥：
   ```javascript
   const CF_TURNSTILE_SITE_KEY = '0x4AAAAAAAXXXXXXXXXXXXXXXXXXXX';  // 替换为你的 Site Key
   const CF_TURNSTILE_SECRET_KEY = '0x4AAAAAAAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX';  // 替换为你的 Secret Key
   ```
4. 点击右上角的 `Deploy`（部署）

## 五、绑定 KV

进入 `Workers & Pages` -> 你的 Worker 名称 -> `Settings`（设置） -> `Bindings`（绑定）：

1. 点击 `Add`（添加绑定）
2. 选择 `KV Namespace`（KV 命名空间）
3. `Variable name` **必须填写 `KV`**（必须大写，代码中写死了这个名字）
4. `KV Namespace`（KV 命名空间） 选择刚才创建的 `TG_BOT_KV`
5. 点击 `Save and deploy`（保存并部署）

## 六、设置环境变量

进入 `Workers & Pages` -> 你的 Worker 名称 -> `Settings`（设置） -> `Variables`（变量）：

在 `Environment Variables`（环境变量）中添加以下变量：

| 变量名 | 类型 | 说明 | 示例 |
|:------:|:----:|:-----|:----:|
| `ENV_BOT_TOKEN` | Secret | 你的 Bot Token | `123456:ABC-DEF...` |
| `ENV_BOT_SECRET` | Secret | Webhook 密钥（随机字符串） | `random_string_123` |
| `ENV_ADMIN_UID` | Plain text | 管理员的 User ID | `123456789` |

> **注意**：`ENV_BOT_TOKEN` 和 `ENV_BOT_SECRET` 建议设置为 `Secret` 类型以保护安全。

## 七、设置 Webhook

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

## ⚠️ 注意事项

1. **KV 绑定名称**：请确保 KV Namespace 的变量名绑定为 `KV`，否则机器人无法记忆状态。
2. **KV 延迟**：Cloudflare KV 存在短暂的最终一致性延迟（约 1 分钟）。如果你刚解封用户，可能需要等几十秒才会生效。
3. **联合封禁**：使用第三方服务查询，会共享用户 ID，请根据隐私需求决定是否开启。
