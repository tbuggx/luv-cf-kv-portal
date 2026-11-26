# 文本文件储存器 CF-Workers-TEXT2KV

CF-Workers-TEXT2KV 是一个在 Cloudflare Workers 上运行的无服务器应用程序,可以将文本文件存储到 Cloudflare Workers KV 键值存储中,并且可以通过 URL 请求读取或更新这些文本文件。它提供了一个安全的方式来管理和访问您的文本文件,同时利用了 Cloudflare 的全球分布式网络。

## 功能特性

- **文本文件存储**: 您可以将任何文本文件存储到 Cloudflare Workers KV 键值存储中,包括纯文本、JSON、XML 等格式。
- **通过 URL 读取文件**: 只需通过构造合适的 URL,就可以读取存储在 KV 中的文本文件内容。
- **通过 URL 更新文件**: 您可以使用 URL 查询参数将新的文本内容上传到 KV,从而实现文件的更新。
- **Base64 编码支持**: 支持使用 Base64 编码的方式上传和下载文件,以应对某些特殊字符场景。
- **安全访问控制**: 通过设置 token 参数,可以限制只有拥有正确密钥的请求才能访问您的文件。
- **辅助工具脚本**: 提供了 Windows 批处理文件和 Linux Shell 脚本,用于方便地从本地上传文件到 KV。

## 使用说明 [部署教程](https://youtu.be/crln7SNrEbQ)

1. **部署到 Cloudflare Workers**

  将项目代码部署到您的 Cloudflare Workers 服务。您需要先在 Cloudflare 上创建一个 Workers 项目,然后将 `worker.js` 文件的内容复制粘贴到 Workers 编辑器中。

2. **创建 KV 命名空间**

  在您的 Cloudflare Workers 项目中,创建一个新的 `KV` 命名空间,用于存储文本文件。记下这个 KV 命名空间的名称,因为您需要将它绑定到 Workers 上。

3. **设置 TOKEN 变量**

  - 为了增加安全性,您需要设置一个 TOKEN 变量,作为访问文件的密钥。在 Cloudflare Workers 的环境变量设置中,添加一个名为 `TOKEN` 的变量,并为其赋予一个安全的值。
  - 默认 TOKEN 为：`passwd`

4. **访问配置页面**

例如 您的workers项目域名为：`txt.cmliussss.workers.dev` , token值为 `passwd`；
  - 访问 `https://您的Workers域名/config?token=您的TOKEN` 或 `https://您的Workers域名/您的TOKEN`，您将看到一个配置页面，其中包含了使用说明和下载脚本的链接。
  - 你的项目配置页则为：
     ```url
     https://txt.cmliussss.workers.dev/config?token=passwd
     或
     https://txt.cmliussss.workers.dev/passwd
     ```

5. **使用辅助脚本上传文件**

  - Windows 用户可以下载 `update.bat` 脚本,然后执行 `update.bat 文件名` 来上传本地文件到 KV。
  - Linux 用户可以下载 `update.sh` 脚本,执行 `./update.sh 文件名` 来上传本地文件。
  - **注意：因为URL长度限制，如果保存内容过长则只能通过直接编辑`KV`对应文件内容来实现大文件的修改保存。**

6. **通过 URL 访问文件**

例如 您的workers项目域名为：`txt.cmliussss.workers.dev` , token值为 `test` , 需要访问的文件名为 `ip.txt`；
  - 构造 URL 的格式为 `https://您的Workers域名/文件名?token=您的TOKEN`。您就可以在浏览器中查看该文件的内容了。
  - 你的访问地址则为： `https://txt.cmliussss.workers.dev/ip.txt?token=test`。

7. **简单的更新文件内容**

  要更新某个文件的内容,可以使用 URL 查询参数 `text` 或 `b64` 来指定新的文本内容或 Base64 编码后的内容。URL 的格式为:
  ```url
  https://您的Workers域名/文件名?token=您的TOKEN&text=新文本内容
  或
  https://您的Workers域名/文件名?token=您的TOKEN&b64=Base64编码的新文本内容
  ```
Workers 会自动将新内容存储到对应的文件中。

通过这个无服务器应用,您可以方便地在 Cloudflare 的分布式网络上存储和管理文本文件,同时享受高性能和安全可靠的优势。欢迎使用 CF-Workers-TEXT2KV!

---

## luvium 定制说明（2026-11-24）

本实例在原版基础上加了 **TOTP 双因素认证** 与 **写白名单**。

### 鉴权

| 接口 | 参数 | 说明 |
|---|---|---|
| 读文件 | `GET /<key>?token=<TOKEN>&otp=<6位码>` | 6 位码默认约 5 分钟有效（`TOTP_WINDOW`） |
| 写 KV | `GET /<key>?token=<TOKEN>&otp=<6位码>&b64=<内容>` | 文件名须在 `WRITE_KEYS` 白名单内 |
| 配置页 | `GET /config?token=<TOKEN>&otp=<6位码>` | 会回显 TOKEN，故同样需要 OTP |

- TOTP：RFC 6238，HMAC-SHA1，6 位，30 秒步长；向前额外接受 `TOTP_WINDOW/步长` 个旧码 → 每个码默认 5 分钟有效
- KV key 只取路径最后一段（`/subscribes/clash.yaml` → `clash.yaml`）

### 环境变量（Cloudflare Pages 项目设置里配）

| 变量 | 类型 | 说明 |
|---|---|---|
| `TOKEN` | plain_text | 固定访问口令 |
| `TOTP_SECRET` | secret_text | Base32 格式的 TOTP 密钥（Microsoft Authenticator 等） |
| `WRITE_KEYS` | plain_text | 允许写入的文件名白名单，逗号分隔；留空=不限制 |
| `READ_REQUIRE_OTP` | plain_text | `1`=读也强制 OTP（默认 `1`）；`0`=读只要 token |
| `TOTP_PERIOD` | plain_text | 步长秒数，默认 `30`（须与 Authenticator 一致） |
| `TOTP_WINDOW` | plain_text | 码的有效时长秒数，默认 `300`（=10 步，5 分钟） |
| `TOTP_SKEW_STEPS` | plain_text | 允许的时钟超前步数，默认 `1` |

### 部署注意

- 本项目为 Pages **Functions** 模式，入口文件必须是 `_worker.js`（不是 `worker.js`）。
- 直接用 Pages API 上传会退化成「拉仓库构建」，必须用 `wrangler pages deploy` 或 GitHub 推送触发。
- `TOTP_SECRET` 用 `wrangler pages secret put TOTP_SECRET --project-name <项目名>` 设置（加密存储）。
