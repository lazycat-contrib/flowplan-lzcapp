# flowplan-lzcapp

[yannicschuller/flowplan](https://github.com/yannicschuller/flowplan) 的懒猫微服（LazyCat）打包。

**Flowplan** —— 开源自托管的 Notion / AppFlowy 替代品：文档、数据库、白板和每日手账。
单容器 + SQLite，不需要 Postgres、Redis 或单独的搜索服务。

## 模式

- **镜像模式**：直接使用上游 GHCR 镜像 `ghcr.io/yannicschuller/flowplan`，由 Action
  转存到 `registry.lazycat.cloud`；版本跟随上游镜像 tag。
- **单实例**：Flowplan 按「恰好一个运行实例」设计，多副本会同时写同一个 SQLite 文件。
- **发布到喵喵（私有）商店**。

## 环境变量与设置向导

安装时的设置向导（`lzc-deploy-params.yml`）暴露常用配置项：

| 分组 | 参数 |
| --- | --- |
| AI | 开关、服务商（ollama / openai / anthropic / gemini）、接口地址、API 密钥、模型、使用范围 |
| 单点登录 | OIDC 签发方、客户端 ID 与密钥、管理员组 |
| 邮件 | SMTP 服务器、端口、用户名、密码、发件地址 |
| 存储 | S3 存储桶、接口地址、访问密钥、区域 |
| 其他 | 每个工作区的存储配额 |

向导里的每一项都是可选的，留空即不设置对应环境变量。

以下几项由平台自动处理，不需要在向导里填：

- `APP_URL` —— 用应用实例域名自动拼成 `https://<实例域名>`
- `FLOWPLAN_DATA_DIR` —— 指向懒猫的持久化目录 `/lzcapp/var/data`
- `FLOWPLAN_TRUSTED_PROXIES=1` —— 微服入口是反向代理
- `FLOWPLAN_ENCRYPTION_KEY` —— 用 `stable_secret` 生成，同一微服同一应用内保持稳定，
  用于加密备份与库内接口密钥

其余环境变量可以在管理界面（Administration）里设置，界面里的值优先。

## 数据与目录

| 容器内路径 | 内容 |
| --- | --- |
| `/lzcapp/var/data` | SQLite 数据库、上传的文件、推送密钥 |

镜像里以 node（uid/gid 1000）运行，`/app/data` 已属于该用户。manifest 里用
`run_as: "1000:1000"` 让系统把 `/lzcapp` 下的持久目录按同一身份映射 owner，
容器可以正常读写挂载点。

> `run_as` 需要 lzcos v1.6.0+，因此 `package.yml` 的 `min_os_version` 是 `1.6.0`。

## 说明

- 第一个注册的账号成为实例管理员，之后可以在管理界面调整实例名称、上传上限、
  是否允许注册等设置
- 局域网内通过 HTTPS 访问；应用需要 `net.internet` 权限获取版本更新提示、
  发送邮件与访问 AI 服务商
- 升级时容器会自行迁移数据库，升级前建议先备份（管理界面 → 版本与备份）

## 所需 Secrets

| Secret | 说明 |
| --- | --- |
| `APPSTORE_URL` / `APPSTORE_TOKEN` | 喵喵商店 API 地址与发布令牌 |
| `APP_ID` | 喵喵商店应用 ID |
| `PRIVATE_STORE_GROUP_CODES` | 私有分组码 |

## 许可

AGPL-3.0（上游）
