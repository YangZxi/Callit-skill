---
name: callit-worker
description: Use when 需要通过已配置的 callit-mcp 按聊天指令创建、搜索、更新或维护 Callit Worker，包括查询 Worker、读取或修改 Worker 文件、更新 Worker 基础信息，或在改动前需要按安全顺序调用对应 MCP tools。
---

# Callit Worker

## 概览

这个 skill 用于通过 `callit-mcp` 管理 Callit 的 Worker。它约束何时该搜索、何时必须先读取现状、何时允许写入，以及哪些操作必须拒绝或先提示用户。

## 何时使用

当用户提出下面这类请求时，使用这个 skill：

- 创建一个新的 Worker
- 通过名称查找已有 Worker
- 查看某个 Worker 的文件列表或文件内容
- 修改 Worker 的代码文件或上传新文件
- 删除 Worker 中的某个文件
- 更新 Worker 的名称、路由、超时时间或启用状态

当请求不涉及 Callit Worker 管理，或不需要 `callit-mcp` 时，不要使用这个 skill。

## 使用前提

先确认宿主环境已经配置名为 `callit-mcp` 的 MCP server。

- 如果 `callit-mcp` 不可用，立即停止执行，并明确提示用户检查 MCP 配置。
- 不要猜测替代的 server 名称。
- 不要在 MCP 不可用时伪造“已创建”或“已更新”的结果。

## 核心工作流

### 1. 先查重或定位目标 Worker

- 创建 Worker 前，先调用 `search_workers`，避免重复创建。
- 用户提到已有 Worker，但名称可能不精确时，也先调用 `search_workers`。
- `search_workers` 使用 `keyword` 参数，不要再传旧字段名 `name`。

更详细的操作顺序见 [references/workflows.md](./references/workflows.md)。

### 2. 修改前先读取现状

涉及文件改动时，默认顺序是：

1. `search_workers`，如果你已经有确定的 Worker ID，可以直接跳过这一步
2. `list_worker_files`
3. `get_worker_file`
4. 再决定 `update_worker_file`、`upload_worker_file` 或 `delete_worker_file`

除非用户明确要求“直接覆盖”，否则不要跳过读取阶段直接写文件。

当改动涉及 Worker 业务代码，尤其是创建或修改 `main.py` / `main.js` 时，在开始编写前先读取 [references/worker_introduction.md](./references/worker_introduction.md)，并按其中引用的 Worker 协议实现代码。

### 3. 按目标选择正确的 tool

- 新建 Worker：`create_worker`
- 更新 Worker 基础信息：`update_worker`
- 新建或覆盖上传文件：`upload_worker_file`
- 修改已有文件内容：`update_worker_file`
- 删除文件：`delete_worker_file`

不要把“更新 Worker 配置”和“更新 Worker 文件”混为一谈。

### 4. 变更后确认结果

- 创建或更新 Worker 后，返回关键字段：`id`、`name`、`runtime`、`route`、`timeout_ms`、`enabled`
- 文件写入或删除后，优先返回最新文件列表；必要时再读取目标文件确认内容
- 如果用户要求总结操作结果，明确说明实际调用了哪些 MCP tools

## 强制约束

- 创建或编写 Worker 代码时，必须遵守 [references/worker_introduction.md](./references/worker_introduction.md) 中定义的入口文件、`handler(ctx)`、stdin 上下文和 stdout 响应契约
- Worker 的代码必须严格按照`runtime`要求编写，请勿使用不受支持的编程语言
- 更新 Worker 时，不要尝试传 `runtime`；`update_worker` 只允许改 `name`、`route`、`timeout_ms`、`enabled`
- 文件名只能是 Worker 根目录下的单层文件名，不支持带 `/` 的路径
- Worker runtime 是不可修改的，所以当碰到用户要求修改 runtime 的情况时，先停下来明确告诉用户这是不允许的，并询问他们是否想删除重建 Worker
- 删除文件前，必须确认目标不是入口文件 `main.py` 或 `main.js`
- `upload_worker_file` 和 `update_worker_file` 都会覆盖已有文件内容，执行前必须意识到这是覆盖写
- 用户要求“新增文件”时，优先使用 `upload_worker_file`
- 用户要求“改现有文件”时，优先使用 `update_worker_file`，若需要的修改的文件不是“文本内容文件”，则使用 `upload_worker_file`进行更新

限制细节见 [references/constraints.md](./references/constraints.md)。

## 引用资料

按需读取这些文件，不要一次性全部展开：

- [references/workflows.md](./references/workflows.md)：创建、更新、读写文件、删除文件的推荐顺序
- [references/tools.md](./references/tools.md)：8 个 MCP tools 的作用、输入和副作用
- [references/constraints.md](./references/constraints.md)：路由、文件名、入口文件、覆盖行为等硬约束
- [references/worker_introduction.md](./references/worker_introduction.md)：已内置在 skill 中的完整 Worker 运行时协议与示例，创建或修改 Worker 代码前必须读取
