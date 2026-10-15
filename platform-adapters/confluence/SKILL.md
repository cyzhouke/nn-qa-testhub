---
name: nn-platform-confluence
description: 通过 mcp-atlassian MCP 访问 Confluence，并使用临时环境变量注入 PAT 作为认证方式，支撑 NN 测试需求、设计说明、版本页面和测试文档读取与维护。
---

# Confluence 平台适配

当 NN 测试工作需要访问 Confluence 页面、空间、评论、附件、需求文档、版本说明或测试报告时使用本适配说明。

## 本机状态

- 状态：`mcp-atlassian` 已安装；临时 PAT + Confluence REST 后端访问方式已验证可读取页面。
- 平台主入口：`mcp-atlassian` MCP，不是 Skill。
- 已验证站点：`https://cf.nn.com`
- 凭证方式：Server/Data Center Personal Access Token，启动 MCP 时通过临时环境变量注入，不写入仓库。
- MCP 安装目录：`%USERPROFILE%\.agents\mcp\mcp-atlassian\`
- 本机可执行文件：`C:\Users\leigod\.agents\mcp\mcp-atlassian\.venv\Scripts\mcp-atlassian.exe`
- 统一启动脚本：`%USERPROFILE%\.agents\mcp\mcp-atlassian\start-confluence-mcp.ps1`
- 多工具接入：Codex 配置在 `%USERPROFILE%\.codex\config.toml`；Claude Code/兼容工具可使用项目级 `.mcp.json`。
- MCP 细节：读取 `references/mcp-atlassian.md`。
- 后端 REST 调用和排障细节：读取 `references/confluence-rest.md`。

## 使用边界

- Confluence 日常操作统一通过 `mcp-atlassian` MCP；临时环境变量 + PAT 是 MCP 的认证方式。
- MCP server 只安装一份到 `.agents`；不同 AI 工具只注册同一个启动脚本，不重复安装。
- MCP 内部会调用 Confluence REST API；直接 REST 调用只作为排障、验证或 MCP 不可用时的兜底。
- 不要把 token 写入仓库、配置文件或日志。
- 默认只读。只有用户明确要求发布、更新或删除页面时，才允许执行写操作。
- 最终回复中不要暴露 page ID、account ID、API token、personal token、OAuth token 或原始存储格式内部字段，除非它们只在工具调用中使用。

## 验证方式

优先验证 MCP server 可启动，并以临时环境变量注入 Confluence 配置：

```powershell
$env:CONFLUENCE_URL = "https://cf.nn.com"
$env:CONFLUENCE_PERSONAL_TOKEN = "<token>"
& "$env:USERPROFILE\.agents\mcp\mcp-atlassian\start-confluence-mcp.ps1"
Remove-Item Env:\CONFLUENCE_PERSONAL_TOKEN -ErrorAction SilentlyContinue
```

测试完成后必须清理当前进程环境变量。需要排障时可按 `references/confluence-rest.md` 直接调用只读 REST 接口确认站点和 PAT 是否可用。

## AI 使用方式

1. 使用 Confluence 前，先读取 `references/mcp-atlassian.md`；需要解析链接或排障时再读取 `references/confluence-rest.md`。
2. 如果用户给的是 `viewpage.action?pageId=...` 链接，解析 `pageId` 后优先通过 `mcp-atlassian` 读取页面；MCP 不可用时才用直接 REST 兜底。
3. 将 Confluence 视为需求、设计依据、版本说明和测试报告的优先事实来源。
4. 基于 Confluence 内容起草测试计划前，先读取目标页面及相关子页面。
5. 摘要中保留页面层级、标题、作者/时间上下文和链接。
6. 更新页面前，先准备清晰的差异或替换章节；只有用户要求发布时才写入。

## QA 场景

- 从需求页面提取验收标准。
- 根据产品和架构页面生成版本测试计划。
- 发布测试报告或复盘记录。
- 对比 Confluence 需求与禅道任务/缺陷状态。
