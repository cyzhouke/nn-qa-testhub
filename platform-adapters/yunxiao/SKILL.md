---
name: nn-platform-yunxiao
description: 通过共享的云效 MCP 访问组织、代码、项目协作、流水线、制品、应用交付和测试管理能力。
---

# 云效平台适配

当 NN 测试工作需要访问云效项目、工作项、代码仓库、流水线、测试用例、测试计划或测试结果时使用本适配说明。

## 本机状态

- MCP 状态：已安装。
- MCP 版本：`alibabacloud-devops-mcp-server` `0.3.64`
- 共享安装目录：`%USERPROFILE%\.agents\mcp\yunxiao-mcp\`
- 启动脚本：`%USERPROFILE%\.agents\mcp\yunxiao-mcp\start-yunxiao-mcp.ps1`
- MCP 服务名：`yunxiao`
- 默认传输：stdio
- 只读访问：已验证组织、用例库、用例目录和具体测试用例详情
- 官方仓库：[https://github.com/aliyun/alibabacloud-devops-mcp-server](https://github.com/aliyun/alibabacloud-devops-mcp-server)

## 凭证配置

不要把令牌写入仓库、MCP 配置文件或聊天记录。运行 AI 工具前，在当前用户环境中配置：

```powershell
$env:YUNXIAO_ACCESS_TOKEN = "<云效个人访问令牌>"
```

云效中心站默认使用：

```text
https://openapi-rdc.aliyuncs.com
```

如果使用 Region 站，再配置：

```powershell
$env:YUNXIAO_API_BASE_URL = "https://<组织专属域名>.devops.aliyuncs.com"
```

可选地限制工具集，减少测试场景的工具数量：

```powershell
$env:DEVOPS_TOOLSETS = "project-management,test-management"
```

不设置 `DEVOPS_TOOLSETS` 时，服务默认启用全部工具集。云效个人令牌的权限应按实际工作范围授予，至少覆盖需要读取的项目协作和测试管理 API。

## 多 AI 工具共用

不同 AI 工具只注册同一个启动脚本，不重复安装 npm 包：

```text
%USERPROFILE%\.agents\mcp\yunxiao-mcp\start-yunxiao-mcp.ps1
```

当前项目的 `.mcp.json` 已登记 `yunxiao`；Codex 全局配置也已登记同一个启动脚本。Claude Code 或其他支持 stdio MCP 的工具，复用上述脚本并从环境变量继承凭证。

## 推荐验证

首次接入后先执行只读操作，例如查询当前组织、项目列表、工作项或测试用例。先验证权限和返回数据，再考虑创建、更新、评论、运行流水线或部署等写操作。

该 MCP 同时包含写入能力。涉及以下操作时，必须根据用户明确授权执行：

- 创建或修改工作项、评论、关联关系
- 创建或修改代码、分支、合并请求
- 运行、停止或重试流水线
- 创建或修改测试用例、测试计划和测试结果
- 创建、修改或执行部署单
