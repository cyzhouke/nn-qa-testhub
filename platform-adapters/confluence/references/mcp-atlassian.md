# mcp-atlassian MCP 接入说明

本文件记录 Confluence 平台使用的 MCP server。它不是 Skill；Skill 只负责指导 AI 何时使用该 MCP。

## 文档来源

- 首页：`https://mcp-atlassian.soomiles.com/docs`
- 安装：`https://mcp-atlassian.soomiles.com/docs/installation.md`
- 认证：`https://mcp-atlassian.soomiles.com/docs/authentication.md`
- 配置：`https://mcp-atlassian.soomiles.com/docs/configuration.md`

文档说明 `mcp-atlassian` 是 Atlassian 产品的 MCP server，可连接 Jira 和 Confluence，支持 Cloud 与 Server/Data Center。Confluence Cloud 完全支持；Confluence Server/Data Center 支持 v6.0+。

## 本机安装

已安装到用户级 `.agents` MCP 目录：

```text
%USERPROFILE%\.agents\mcp\mcp-atlassian\
```

本机实际可执行文件：

```text
C:\Users\leigod\.agents\mcp\mcp-atlassian\.venv\Scripts\mcp-atlassian.exe
```

安装方式：

```powershell
$root = "$env:USERPROFILE\.agents\mcp\mcp-atlassian"
$venv = "$root\.venv"
New-Item -ItemType Directory -Force -Path $root | Out-Null
uv venv $venv
uv pip install --python "$venv\Scripts\python.exe" mcp-atlassian
& "$venv\Scripts\mcp-atlassian.exe" --version
```

当前验证结果：

```text
mcp-atlassian, version 0.23.1
```

## 统一启动脚本

后续多 AI 工具共用同一个启动脚本：

```text
%USERPROFILE%\.agents\mcp\mcp-atlassian\start-confluence-mcp.ps1
```

该脚本负责：

- 检查 `mcp-atlassian.exe` 是否存在。
- 检查 `CONFLUENCE_PERSONAL_TOKEN` 是否存在。
- 设置 `CONFLUENCE_URL=https://cf.nn.com`。
- 设置只读模式和默认工具集。
- 启动 `%USERPROFILE%\.agents\mcp\mcp-atlassian\.venv\Scripts\mcp-atlassian.exe --read-only`。

## MCP 客户端配置示例

将以下配置加入支持 MCP 的客户端。不要把真实 token 写入仓库。

Codex/ChatGPT：

```toml
[mcp_servers.mcp-atlassian]
type = "stdio"
command = "powershell"
args = ["-NoProfile", "-ExecutionPolicy", "Bypass", "-File", "C:\\Users\\leigod\\.agents\\mcp\\mcp-atlassian\\start-confluence-mcp.ps1"]
startup_timeout_sec = 120
env_vars = ["CONFLUENCE_PERSONAL_TOKEN"]
```

Claude Code / 项目级 `.mcp.json`：

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "type": "stdio",
      "command": "powershell",
      "args": [
        "-NoProfile",
        "-ExecutionPolicy",
        "Bypass",
        "-File",
        "${USERPROFILE}\\.agents\\mcp\\mcp-atlassian\\start-confluence-mcp.ps1"
      ],
      "env": {
        "CONFLUENCE_PERSONAL_TOKEN": "${CONFLUENCE_PERSONAL_TOKEN}"
      }
    }
  }
}
```

## 认证方式

当前 `https://cf.nn.com` 使用 Server/Data Center Personal Access Token：

```text
CONFLUENCE_URL=https://cf.nn.com
CONFLUENCE_PERSONAL_TOKEN=your_personal_access_token
```

如内部证书不是系统信任链的一部分，可按官方文档评估 `CONFLUENCE_SSL_VERIFY=false`。不要默认关闭 SSL 验证。

## 安全约束

- 不要把 Confluence token、OAuth secret、cookie 或 session 文件写入仓库。
- `CONFLUENCE_PERSONAL_TOKEN` 当前保存为 Windows 用户级环境变量；工具配置只引用变量名，不存明文。
- 默认启用只读模式；只有用户明确要求写入 Confluence 时，才允许取消 `--read-only` 或 `READ_ONLY_MODE=true`。
- 推荐限制工具集为 `TOOLSETS=default`；需要 Jira agile、legacy 或全量工具时再显式开启。
- 如需限制搜索范围，使用 `CONFLUENCE_SPACES_FILTER` 指定空间 key。
- 如果 MCP 客户端已经配置了同名 server，先告知用户现有配置并确认是否覆盖或复用。

## 可用方向

优先使用 Confluence 相关工具集：

- 页面读取、创建、更新、删除和页面树导航。
- CQL 搜索和用户查找。
- 附件上传、下载、列表和管理。
- 评论、标签、页面分析和权限检查。
- 模板列表和基于模板创建页面。
