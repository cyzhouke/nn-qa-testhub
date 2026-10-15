# NN QA Testhub

这是一个面向 NN 日常测试工作的 AI 工作台。目标不是把所有平台逻辑写死在一个提示词里，而是用项目内 Skill、CLI、MCP 入口把企业微信、飞书、Confluence、禅道等测试协作平台聚合起来。

## 推荐结构

- `skills/nn-test-workbench/SKILL.md`: 工作台总入口，告诉 AI 如何判断任务、选择平台、沉淀测试产物。
- `skills/nn-test-workbench/references/platform-routing.md`: 平台路由规则，避免 AI 在多个系统之间乱跳。
- `platform-adapters/SKILL.md`: 平台适配目录入口，说明目录用途和平台路由。
- `platform-adapters/<platform>/SKILL.md`: 每个平台一个适配说明，记录可用 Skill/CLI/MCP/API、本机路径、认证检查和安全边界。
- `registry/platforms.yaml`: 当前平台接入状态登记表，便于快速确认哪个平台已可用。

## 当前接入状态

| 平台         | 状态                 | 推荐入口                                        |
| ---------- | ------------------ | ------------------------------------------- |
| 企业微信       | 已接入                | `.agents\skills\wecom-unified` 企业微信能力 Skill |
| 飞书         | 待安装/配置             | Lark/Feishu CLI 或官方远程 MCP                   |
| Confluence | MCP 已安装，REST 认证已验证 | 共享 `mcp-atlassian` 启动脚本                     |
| 禅道         | Skill 已安装，CLI 待安装 | `.agents\skills\zentao-cli`；运行时命令为 `zentao` |
| 云效         | MCP 已安装，只读访问已验证 | `.agents\mcp\yunxiao-mcp`；共享启动脚本 |

## 使用方式

当需要让 AI 做测试工作时，优先让它读取：

1. `skills/nn-test-workbench/SKILL.md`
2. 如果涉及 CLI 安装、升级或修复，读取 `skills/nn-test-workbench/references/cli-install-policy.md`
3. `platform-adapters/SKILL.md`
4. 对应平台的 `platform-adapters/<platform>/SKILL.md`
5. `registry/platforms.yaml`

平台 Skill 中如果写有本机路径，以本机路径为准；如果只有安装说明，则先完成安装、认证和只读验证，再执行写操作。禅道 Skill 已安装到 `%USERPROFILE%\.agents\skills\zentao-cli\`，但 `zentao` CLI 本体仍需按 CLI 安装策略单独处理。

## CLI 安装限制

- 安装任何 CLI 前，必须先扫描本机是否已存在相同命令。
- 如果已安装，先告诉用户检测到的路径和版本，并询问是否继续使用已安装 CLI。
- 如果未安装，统一安装到 `%USERPROFILE%\.agents\` 下；npm CLI 使用 `%USERPROFILE%\.agents\npm-global\`。
- 不静默覆盖已有命令，不静默修改系统 PATH。

## MCP 共用原则

- MCP server 只安装一次，统一放在 `%USERPROFILE%\.agents\mcp\`。
- Codex、Claude Code 或其他 AI 工具只注册同一个 MCP 启动脚本，不重复安装。
- Confluence 当前共用启动脚本：`%USERPROFILE%\.agents\mcp\mcp-atlassian\start-confluence-mcp.ps1`。
- 云效当前共用启动脚本：`%USERPROFILE%\.agents\mcp\yunxiao-mcp\start-yunxiao-mcp.ps1`。
- 凭证通过本机环境变量或凭证管理器提供，不写入仓库。
