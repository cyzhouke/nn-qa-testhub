# CLI 安装策略

当用户要求安装、升级、修复或接入某个 CLI 时，必须遵循本策略。

## 安装根目录

统一安装到用户级 `.agents` 目录下，不写入系统级全局目录：

```text
%USERPROFILE%\.agents\
```

推荐子目录：

- npm CLI：`%USERPROFILE%\.agents\npm-global\`
- 独立二进制 CLI：`%USERPROFILE%\.agents\cli\<cli-name>\`
- CLI 配置和凭证缓存：优先使用工具自身支持的安全位置；不得写入仓库。

## 安装前扫描

安装前必须先扫描本机是否已经存在相同 CLI。

PowerShell 示例：

```powershell
Get-Command <command-name> -ErrorAction SilentlyContinue | Select-Object -ExpandProperty Source
Get-ChildItem -LiteralPath "$env:USERPROFILE\.agents" -Recurse -File -ErrorAction SilentlyContinue |
  Where-Object { $_.Name -match '^(<command-name>|<command-name>\.cmd|<command-name>\.ps1|<command-name>\.exe)$' } |
  Select-Object -ExpandProperty FullName
```

如果平台适配说明中记录了历史路径，也要检查该路径是否仍然存在。

## 已安装时

如果扫描发现本机已安装相同 CLI：

1. 不要直接重装、覆盖或修改 PATH。
2. 向用户说明已发现的 CLI 路径和可获取的版本号。
3. 询问用户是否继续使用已安装的 CLI，或者是否需要升级/修复/另装一份到 `.agents`。
4. 只有用户确认后，才继续使用该 CLI 或执行安装/升级。

## 未安装时

如果未发现相同 CLI，才安装到 `.agents` 下。

npm CLI 推荐命令：

```powershell
$prefix = "$env:USERPROFILE\.agents\npm-global"
New-Item -ItemType Directory -Force -Path $prefix | Out-Null
npm install --global --prefix $prefix <npm-package>
& "$prefix\<command-name>.cmd" --version
```

独立二进制 CLI 推荐放到：

```text
%USERPROFILE%\.agents\cli\<cli-name>\
```

安装后必须记录：

- CLI 名称
- 安装路径
- 版本号
- 验证命令
- 认证状态或下一步认证方式

这些信息优先更新到 `registry/platforms.yaml` 和对应 `platform-adapters/<platform>/SKILL.md`。

## 安全边界

- 不要把 token、密码、refresh token、cookie 或 session 文件写入本仓库。
- 不要静默修改系统 PATH。若某个 CLI 需要 PATH，先给出临时调用路径；只有用户要求时才说明如何加入 PATH。
- 不要为了安装 CLI 覆盖已有同名命令。已有命令必须先告知用户并等待确认。
