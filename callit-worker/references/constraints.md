# Callit Worker 约束

## MCP 依赖

- 这个 skill 依赖已配置好的 `callit-mcp`
- 若 `callit-mcp` 不可用，必须停止并提示用户检查 MCP 配置

## Worker 基础信息

- `create_worker` 需要完整的 `name`、`runtime`、`route`、`timeout_ms`
- `update_worker` 只能修改 `name`、`route`、`timeout_ms`、`enabled`
- 不要尝试通过 `update_worker` 修改 `runtime`

## Worker 代码契约

- 创建或修改 Worker 代码前，先读取 [worker_introduction.md](./worker_introduction.md)
- Python Worker 必须使用 `main.py`，并定义可调用的 `handler(ctx)`
- Node Worker 必须使用 `main.js`，并导出可调用的 `handler`
- 代码要按文档中的 `ctx.request`、`ctx.event` 结构读取输入
- 代码输出必须符合文档中的 stdout JSON 契约，至少返回合法的 `body`，并正确使用 `status`、`headers`、`file`
- 如果修改入口文件逻辑，不能脱离文档约定自定义一套输入输出协议

## 路由规则

- `route` 必须以 `/` 开头
- 通配符仅支持结尾 `/*`
- 路由不明确时，不要自行补全复杂规则，先向用户确认

## 文件规则

- 只支持 Worker 根目录单层文件名
- `filename` 不能包含 `/`
- 修改文件前，默认应先读取当前文件内容
- `upload_worker_file` 与 `update_worker_file` 都会覆盖已有内容

## 删除规则

- 删除前先确认文件存在
- 删除前先确认不是入口文件
- `main.py` 与 `main.js` 不能删除

## 安全行为

- 不要在未读取现状的情况下盲目覆盖用户文件，除非用户明确要求直接覆盖
- 搜索结果不唯一时，先让用户确认目标 Worker
- 不要声称某次创建、修改或删除已经成功，除非 MCP 返回成功结果
