---
name: nn-platform-feishu
description: 在本地凭证配置完成后，通过飞书/Lark 官方 CLI、MCP 或开放 API 支持 NN 测试协作。
---

# 飞书平台适配

当 NN 测试工作需要访问飞书/Lark 聊天、文档、表格、任务、会议或机器人通知时使用本适配说明。

## 本机状态

- 状态：当前 PATH 未检测到可用命令。
- 预期 CLI 命令：`lark-cli`
- 检测到的路径：无。
- 推荐入口：在工作区完成应用授权后，使用官方 Lark/Feishu CLI 或官方 MCP。
- 已核验 npm 包：`@larksuite/cli` 版本 `1.0.96`，包说明为 Lark/Feishu 开放平台官方 CLI。

## 安装方式

安装 `lark-cli` 前，必须先遵循 `skills/nn-test-workbench/references/cli-install-policy.md`：扫描本机是否已存在 `lark-cli`，如果已存在，先告知路径和版本，并询问是否继续使用。

如果本机未发现可用版本，统一安装到 `.agents` 下：

```powershell
$prefix = "$env:USERPROFILE\.agents\npm-global"
New-Item -ItemType Directory -Force -Path $prefix | Out-Null
npm install --global --prefix $prefix @larksuite/cli
& "$prefix\lark-cli.cmd" --version
```

如果官方 CLI 包名或安装方式发生变化，安装前先查询当前飞书/Lark 开放平台文档。

MCP 路径：

1. 在公司飞书工作区创建或复用一个应用。
2. 只授予测试工作需要的最小权限，例如文档读取、表格读写、消息发送或任务读写。
3. 使用应用凭证配置官方远程 MCP，或配置本地 MCP server wrapper。
4. 在 Codex 或宿主 Agent 环境中注册 MCP server。

## 必要配置

凭证必须放在仓库外部：

```powershell
$env:FEISHU_APP_ID = "<app-id>"
$env:FEISHU_APP_SECRET = "<app-secret>"
```

如果使用 OAuth 或用户授权，将 refresh token 存放到平台支持的安全凭证存储中，不写入项目文件。

## 验证方式

写操作前先做只读验证：

```powershell
lark-cli --version
```

然后执行一个无副作用读取，例如读取当前用户、机器人身份，或一个已知测试文档。验证阶段不要发送消息或修改文档。

## AI 使用方式

1. 先检查 `registry/platforms.yaml`，确认本机配置的命令或 MCP server 名称。
2. 如果 MCP 已提供文档、表格、聊天和任务的结构化工具，优先使用 MCP。
3. 如果只有 CLI/API 可用，按项目 wrapper 或官方命令文档构造参数。
4. 最终回复中不要暴露 app secret、token、open ID、chat ID、文档 token 或 cursor。
5. 无法从用户请求推断目标文档、会话或任务时，先询问目标。

## QA 场景

- 读取飞书需求文档并提取验收标准。
- 从文档生成测试用例，并发布到飞书表格。
- 汇总与缺陷或版本相关的聊天决策。
- 通过批准的机器人渠道发送 QA 进度通知。
