# Authright Sharing Plugin

通过 Codex 查询和分享团队链接。当前 Plugin 版本：**0.2.0**。

服务地址：[sharing.authright.com](https://sharing.authright.com)；MCP 地址：`https://sharing.authright.com/mcp`。Web、Worker、数据库和内容处理 API 已在服务器运行，使用本 Plugin 无需在电脑启动本地服务或 Docker。

本仓库保存 Plugin 文件和安装说明；应用服务源码位于 [Authright_Sharing](https://github.com/TAO44123/Authright_Sharing)。本版面向 Codex，Claude Code / Cursor 适配尚未提供。生产端点可达与 OAuth 发现已检查；从本 Git 来源的干净安装、生产授权、真实工具调用、更新和回退仍需验收。

## 安装

前提：Codex 支持 Plugin marketplace，安装者具有本仓库读取权限，并使用 `@authright.com` 公司 Google 账号完成 Sharing 授权。私有仓库需要先配置 GitHub HTTPS 或 SSH 读取权限。

在终端添加这个仓库作为目录源并安装：

```sh
codex plugin marketplace add TAO44123/Authright_Sharing_Plugin --ref main
codex plugin add sharing@authright-sharing
```

若使用 GitHub SSH，可将第一条替换为：

```sh
codex plugin marketplace add git@github.com:TAO44123/Authright_Sharing_Plugin.git --ref main
```

也可以先克隆到电脑，再登记本地目录源：

```sh
git clone https://github.com/TAO44123/Authright_Sharing_Plugin.git
codex plugin marketplace add ./Authright_Sharing_Plugin
codex plugin add sharing@authright-sharing
```

CLI 命令来自当前已安装 Codex 的帮助与 [OpenAI 官方 Plugin 文档](https://developers.openai.com/plugins/build/plugins)。界面和管理员策略可能因客户端版本不同而变化，以上 Git 来源安装尚待真实验收。

安装后在 Codex 插件页面确认 Sharing 版本为 `0.2.0`，连接地址为 `https://sharing.authright.com/mcp`，再打开新的聊天。

### 已安装本地开发版的用户

旧目录源为 `sharing-local`；本仓库目录源为 `authright-sharing`。在切换时关闭旧来源下的 Sharing，再启用新来源下的 Sharing，避免两个同名 MCP 连接同时工作。本次上传不会自动修改你电脑上已安装的插件或授权。

## 登录与授权

1. 从 Codex 的 Sharing 插件连接设置发起 OAuth 登录。
2. 在浏览器使用公司 Google 账号登录生产 Sharing。
3. 核对当前账号和权限，点击 **Allow access**。
4. 返回 Codex，在新的聊天中执行一次只读查询确认连接。

授权范围包括 `shares:read`、`shares:write`、`members:read`；需要刷新令牌时使用 `offline_access`。本地环境的凭据不直接适用于生产环境。客户端授权回跳可能使用 `127.0.0.1` 临时端口，这是客户端接收授权码的地址，MCP 服务仍在生产域名。

如果客户端未打开授权页面，可在终端显式发起生产 OAuth：

```sh
codex -c 'mcp_servers.sharing.url="https://sharing.authright.com/mcp"' \
  mcp login sharing \
  --scopes shares:read,shares:write,members:read,offline_access \
  --oauth-client-registration dcr
```

该命令只覆盖本次 CLI 配置，不会更新安装包；CLI 登录成功也不能代替桌面聊天的实际工具调用验证。

## 六个指令入口

| 指令 | 功能 |
| --- | --- |
| `$sharing:sharing` | 综合入口，根据需求组合查询、成员检索、提交和撤回操作。 |
| `$sharing:list-shares` | 查询团队分享；支持时间、成员、关键词筛选与分页，默认最近七天。 |
| `$sharing:get-share` | 查看一条分享的链接、分享人、处理状态、文章/视频摘要或视频作者描述。 |
| `$sharing:list-members` | 展示或搜索团队成员，按姓名/邮箱区分同名用户。 |
| `$sharing:share-link` | 将用户指定的文章或 YouTube URL 保存为团队分享。 |
| `$sharing:withdraw-share` | 撤回当前用户自己发布的指定分享。 |

示例：

```text
$sharing:list-shares 最近七天团队分享了什么？
$sharing:list-members 展示所有成员列表
$sharing:get-share 查看刚才那条分享的详情
$sharing:share-link 将这个链接分享到团队：https://example.com/article
```

六个入口对应一个综合 Skill 与五个操作 Skill；实际能力由五个 MCP 工具提供。保存成功不等于摘要已生成。历史视频摘要可能仅覆盖音轨，新版可能覆盖语音与采样画面；以返回的 `content_scope_note` 为准。摘要和作者 Description 不代表文章全文或完整视频转录。

## 更新与回退

通过 Git marketplace 安装的用户，先刷新目录快照：

```sh
codex plugin marketplace upgrade authright-sharing
```

随后在插件页面更新或重新安装 Sharing，核对版本与连接地址，并打开新聊天。通过本地 clone 安装时，先在该 clone 中执行 `git pull --ff-only`，再按客户端支持的方式更新或重新安装。不要直接修改 Codex 的安装缓存。

初次上传仅提供 `0.2.0`，目前没有已验收的 Git 发布标签或可回退版本。后续发布记录确切提交与兼容范围；回退到旧版时再按对应提交/标签登记目录并重新安装，不覆盖既有版本标签。客户端实际更新和回退流程仍待验收。

## 卸载与撤销连接

在 Codex 插件页面移除 Sharing；如还需撤销数据访问权，进入 [Sharing 账户页](https://sharing.authright.com/account) 撤销对应 Agent 连接。卸载插件与服务端撤权是两步操作，不能假定卸载会自动撤销 OAuth。

## 文件结构

```text
.agents/plugins/marketplace.json
plugins/sharing/
  .codex-plugin/plugin.json
  .mcp.json
  skills/
    sharing/SKILL.md
    list-shares/SKILL.md
    get-share/SKILL.md
    list-members/SKILL.md
    share-link/SKILL.md
    withdraw-share/SKILL.md
README.md
CHANGELOG.md
```

marketplace 中的 `source: local` 表示 Plugin 位于已获取的目录仓库内部；Git 仓库本身通过 `marketplace add` 获取。这个字段不意味着 MCP 使用本地服务器。

本安装包包含连接地址与 Skill，不含用户数据、服务端源码、API Key、Google Client Secret、数据库凭据或个人令牌。
