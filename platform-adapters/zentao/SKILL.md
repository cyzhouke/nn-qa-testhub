---
name: nn-platform-zentao
description: 通过 MCP、API 或 CLI 访问禅道中的 NN 测试需求、任务、缺陷、构建、发布和测试执行状态。
---

# 禅道平台适配

当 NN 测试工作需要访问禅道需求、任务、缺陷、测试用例、测试套件、构建、发布或执行状态时使用本适配说明。

## 本机状态

- Skill 状态：已安装并通过 Codex Skill 校验。
- Skill 路径：`%USERPROFILE%\.agents\skills\zentao-cli\SKILL.md`
- Skill 来源：`https://github.com/easysoft/zentao-skills`
- Skill 版本：`0.3.1`
- CLI 状态：当前未检测到 `zentao` 命令，也未检测到 `zentao-cli` npm 包。
- 推荐入口：先使用已安装的 `zentao-cli` Skill；实际访问需要先准备 `zentao` CLI 和禅道凭证。

## 安装方式

### Skill

本机已统一安装到全局共享目录：

```text
%USERPROFILE%\.agents\skills\zentao-cli\
```

它不是项目内副本，也不是 `.codex` 专属 Skill，其他支持本地 Skill 的 AI 工具可以复用该目录中的内容。

### CLI

安装 `zentao` 前，必须先遵循 `skills/nn-test-workbench/references/cli-install-policy.md`：扫描本机是否已存在 `zentao`，如果已存在，先告知路径和版本，并询问是否继续使用。

如果本机未发现可用版本，才统一安装到 `.agents` 下：

```powershell
$prefix = "$env:USERPROFILE\.agents\npm-global"
New-Item -ItemType Directory -Force -Path $prefix | Out-Null
npm install --global --prefix $prefix zentao-cli
& "$prefix\zentao.cmd" --version
```

安装后以 CLI 自带帮助为准：

```powershell
& "$env:USERPROFILE\.agents\npm-global\zentao.cmd" --version
& "$env:USERPROFILE\.agents\npm-global\zentao.cmd" help
& "$env:USERPROFILE\.agents\npm-global\zentao.cmd" profile --format=json
```

## 必要配置

凭证必须放在仓库外部：

```powershell
$env:ZENTAO_URL = "https://zentao.example.com"
$env:ZENTAO_ACCOUNT = "<account>"
$env:ZENTAO_TOKEN = "<token>"
# 或使用 ZENTAO_PASSWORD；Token 优先于密码
```

也可以让 CLI 通过交互式 `zentao login` 管理本地 Profile。默认配置文件由 CLI 管理，通常位于：

```text
%USERPROFILE%\.config\zentao\zentao.json
```

## 验证方式

先执行只读检查：

```powershell
zentao --version
zentao help
zentao profile --format=json
```

凭证准备好后，再执行一个用户授权范围内的只读查询，例如产品、项目、需求、任务或 Bug 列表。不要仅凭 `profile` 成功就断言服务连接和业务权限正常。

## AI 使用方式

1. 将禅道视为缺陷、任务、需求状态和测试执行跟踪的事实来源。
2. 用户可见输出优先使用公开编号、标题、产品名、模块名、优先级、严重程度和状态。
3. 不在最终回复中暴露 session token、非公开内部数字 ID 或 API 游标。
4. 修改状态、评论、指派或编辑缺陷前，先总结拟写入内容；如果用户请求已经明确且无歧义，可以直接执行。
5. 可用时，将测试用例和缺陷分析追溯到需求或 Confluence 页面。

## QA 场景

- 读取某个版本的开放缺陷并汇总风险。
- 根据变更需求和缺陷生成回归范围。
- 将复现记录整理成禅道缺陷草稿。
- 在测试轮次结束后更新测试执行状态。

详细命令参数、数据输出和写入边界以全局 Skill 的参考资料为准：

- `%USERPROFILE%\.agents\skills\zentao-cli\references\data-output.md`
- `%USERPROFILE%\.agents\skills\zentao-cli\references\writes.md`
