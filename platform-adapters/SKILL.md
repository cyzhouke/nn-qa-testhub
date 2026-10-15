---
name: platform-adapters
description: 说明 NN 测试工作台的平台适配目录用途，并路由 AI 到企业微信、飞书、Confluence、禅道等具体平台适配说明；本目录不是 CLI、MCP 或全局 Skill 的安装位置。
---

# 平台适配目录

`platform-adapters/` 是 NN 测试工作台的平台接入说明集合。它记录每个平台应该通过 Skill、CLI、MCP、REST API 或其他 wrapper 如何访问，以及本地已验证路径、凭证边界和安全规则。

本目录不是安装目录：

- 全局 AI Skill 安装在 `%USERPROFILE%\.agents\skills\`。
- MCP server 安装在 `%USERPROFILE%\.agents\mcp\`。
- npm CLI 运行入口安装在 `%USERPROFILE%\.agents\npm-global\`。

## 使用方式

访问外部平台前，先选择并读取对应平台适配说明：

- 企业微信：`platform-adapters/wecom/SKILL.md`
- 飞书：`platform-adapters/feishu/SKILL.md`
- Confluence：`platform-adapters/confluence/SKILL.md`
- 禅道：`platform-adapters/zentao/SKILL.md`
- 云效：`platform-adapters/yunxiao/SKILL.md`

如果不确定应该使用哪个平台，先读取 `skills/nn-test-workbench/references/platform-routing.md`。

## 编写原则

- 顶层只负责路由和边界，不重复各平台的详细命令。
- 平台目录内的 `SKILL.md` 记录该平台的默认入口、验证方式和安全边界。
- 复杂安装、认证、MCP 配置、REST 参数等细节放在该平台的 `references/` 下。
- 新增平台时，同步更新 `registry/platforms.yaml`、`README.md` 和 `00-NN测试工作台.md`。
- 不把 token、密码、cookie、session、refresh token 或真实敏感配置写入仓库。
