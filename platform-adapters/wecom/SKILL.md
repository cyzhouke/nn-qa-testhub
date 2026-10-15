---
name: nn-platform-wecom
description: 通过企业微信支持 NN 测试协作，包括通讯录、文档、表格、会议、日程、待办、微盘、邮件和消息等场景。
---

# 企业微信平台适配

当 NN 测试工作需要访问或操作企业微信数据时使用本适配说明，例如读取聊天上下文、搜索文档、创建日程、查询待办、发送通知，或处理企业微信文档和智能表格。

## 本机状态

- 状态：企业微信能力 Skill 已统一安装到 `.agents`，运行时 CLI 已授权。
- 企业微信能力 Skill：`%USERPROFILE%\.agents\skills\wecom-unified\SKILL.md`
- 本机实际 Skill 路径：`C:\Users\leigod\.agents\skills\wecom-unified\SKILL.md`
- Skill 目录说明：`wecom-unified` 是 AI 要读取的企业微信能力目录，里面包含 `SKILL.md`、`references` 和 `scripts`。
- CLI 命令：`wecom-cli`
- 运行时 CLI 入口：`%USERPROFILE%\.agents\npm-global\wecom-cli.cmd`
- 本机实际 CLI 入口：`C:\Users\leigod\.agents\npm-global\wecom-cli.cmd`
- CLI 入口说明：`.cmd/.ps1` 是 npm 在 Windows 上自动生成的命令入口，不是企业微信能力 Skill 目录。
- 检测到的 CLI 版本：`wecom-cli 1.3.2`
- 旧检测 CLI 路径：`C:\Users\leigod\AppData\Roaming\npm\wecom-cli.ps1`
- 说明：企业微信能力只以 `.agents\skills\wecom-unified` 为主入口。

## 官方资料

- 企业微信 CLI 官方文档：`https://open.work.weixin.qq.com/help2/pc/21676`
- 安装、升级或排查 `wecom-cli` 前，优先查看官方文档；如果官方文档与本文件存在冲突，以官方文档和当前本机检测结果为准。

## 验证方式

如果当前会话尚未验证过企业微信访问能力，先运行：

```powershell
$wecomCli = "$env:USERPROFILE\.agents\npm-global\wecom-cli.cmd"
& $wecomCli --version
& $wecomCli auth show --status
```

仅当版本不低于 `1.1.0`，且认证状态为 `authorized` 时，才继续执行业务操作。

## 安装或修复

安装或修复企业微信能力前，先区分两件事：

- 企业微信能力 Skill：安装到 `%USERPROFILE%\.agents\skills\wecom-unified\`。
- npm 运行时 CLI：由 npm 生成 `.cmd/.ps1` 入口，放到 `%USERPROFILE%\.agents\npm-global\`。

修复 `wecom-cli` 前，必须先遵循 `skills/nn-test-workbench/references/cli-install-policy.md`：扫描本机是否已存在 `wecom-cli`。如果发现其他位置已有 `wecom-cli`，应告知用户路径和版本；除非用户明确要求继续使用旧路径，否则优先使用 `.agents` 下的运行时 CLI：

```powershell
$prefix = "$env:USERPROFILE\.agents\npm-global"
New-Item -ItemType Directory -Force -Path $prefix | Out-Null
npm install --global --prefix $prefix @wecom/cli
& "$prefix\wecom-cli.cmd" --version
& "$prefix\wecom-cli.cmd" auth init --noninteractive
& "$prefix\wecom-cli.cmd" auth show --status
```

认证初始化可能输出二维码或授权链接，并等待用户完成扫码/授权。

## AI 使用方式

1. 执行任何真实企业微信操作前，只读取 `%USERPROFILE%\.agents\skills\wecom-unified\SKILL.md`，并遵循其中的业务路由表。
2. 只读取当前任务需要的 reference 文件，例如通讯录、文档、会议、消息、日程、智能表格或待办。
3. 按已加载 reference 中的参数规范调用 `%USERPROFILE%\.agents\npm-global\wecom-cli.cmd`。
4. 最终回复中不要暴露内部 ID。优先使用可读名称、标题、时间、文档名或链接。
5. 发送消息、编辑文档、创建会议、修改待办等写操作，必须确认用户请求中已经明确要求执行。

## QA 场景

- 写测试状态报告前，搜索近期测试讨论。
- 按可读姓名查找测试、产品或研发联系人。
- 创建或更新 QA 日程、会议和会议纪要。
- 读取或发布企业微信文档、智能表格中的测试记录。
- 向已知近期会话发送版本测试通知。
