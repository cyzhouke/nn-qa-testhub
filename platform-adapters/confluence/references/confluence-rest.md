# Confluence REST 后端与排障说明

Confluence 后续统一通过 `mcp-atlassian` MCP 操作。本文记录 MCP 背后的 Confluence REST API 认证和排障方式：启动 MCP 时用临时环境变量注入 Personal Access Token；当 MCP 不可用或需要定位问题时，可临时直接调用只读 REST API 验证站点和凭证。不要把 token 写入仓库、配置文件、日志或最终回复。

## 已验证信息

- 站点：`https://cf.nn.com`
- 部署类型：Confluence Server/Data Center 风格。
- 凭证方式：Personal Access Token。
- 已验证只读接口：`GET /rest/api/content/{pageId}?expand=body.storage,version,space,history`
- 日常入口：`mcp-atlassian` MCP。
- 直接 REST：仅用于排障、验证或 MCP 不可用时的兜底。

## 安全规则

- token 只在当前命令进程或当前 PowerShell 进程中临时设置。
- 命令结束后执行 `Remove-Item Env:\CONFLUENCE_PERSONAL_TOKEN -ErrorAction SilentlyContinue`。
- 不在最终回复中复述 token。
- 不把 token 写入 `.env`、YAML、JSON、Markdown、shell profile 或 MCP 客户端配置。
- 默认只读；写操作必须来自用户明确请求。

## 页面 URL 解析

用户给出如下 URL 时：

```text
https://cf.nn.com/pages/viewpage.action?pageId=132650198
```

从 query string 中提取 `pageId`，再调用：

```text
https://cf.nn.com/rest/api/content/132650198?expand=body.storage,version,space,history
```

## 读取页面

优先通过 `mcp-atlassian` MCP 读取页面。以下 PowerShell + Python 示例仅用于排障或 MCP 不可用时的只读兜底。运行时由用户临时提供 token，不要硬编码到文件中。

```powershell
$env:CONFLUENCE_PERSONAL_TOKEN = "<token>"
$py = @'
import os, re
import requests
from bs4 import BeautifulSoup

base = "https://cf.nn.com"
page_id = "<pageId>"
token = os.environ["CONFLUENCE_PERSONAL_TOKEN"]
url = f"{base}/rest/api/content/{page_id}"
params = {"expand": "body.storage,version,space,history"}
headers = {"Authorization": f"Bearer {token}", "Accept": "application/json"}

resp = requests.get(url, headers=headers, params=params, timeout=30)
resp.raise_for_status()
data = resp.json()
html = data.get("body", {}).get("storage", {}).get("value", "")
soup = BeautifulSoup(html, "html.parser")
for tag in soup(["script", "style"]):
    tag.decompose()
text = soup.get_text("\n")
text = re.sub(r"\n{3,}", "\n\n", text)
text = re.sub(r"[ \t]+", " ", text).strip()

print("TITLE", data.get("title", ""))
print("SPACE", data.get("space", {}).get("name", ""))
print("VERSION", data.get("version", {}).get("number", ""))
print("CREATOR", data.get("history", {}).get("createdBy", {}).get("displayName", ""))
print("TEXT_BEGIN")
print(text)
print("TEXT_END")
'@
python -c $py
Remove-Item Env:\CONFLUENCE_PERSONAL_TOKEN -ErrorAction SilentlyContinue
```

## 写入页面

优先通过 `mcp-atlassian` MCP 写入页面。只有用户明确要求修改 Confluence 页面时才执行写操作。写入前必须：

1. 先读取页面当前内容和版本号。
2. 向用户说明要替换或追加的章节/内容。
3. 通过 MCP 或 Confluence REST API 更新页面，并将 `version.number` 加 1。
4. 写入后再次读取页面确认标题、版本和关键内容。

不要在未确认页面版本的情况下直接写入，避免覆盖他人更新。

## 输出规则

- 可以展示页面标题、空间名、版本号、创建人、页面链接和正文摘要。
- 不要展示 token、内部 account ID、权限响应中的内部标识。
- 用户未要求全文时，优先给结构化摘要；用户要求全文时再展示提取后的正文。
