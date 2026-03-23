# Callit Worker 约束

## MCP 依赖

- 这个 skill 依赖已配置好的 `callit-mcp`
- 若 `callit-mcp` 不可用，必须停止并提示用户检查 MCP 配置

## Worker 基础信息

- `update_worker` 不能修改 `runtime` 信息
- 不要尝试通过 `update_worker` 修改 `runtime`

## Worker 代码契约

- 创建或修改 Worker 代码前，先读取 [worker_introduction.md](./worker_introduction.md)
- Python Worker 必须使用 `main.py`，并定义可调用的 `handler(ctx)`
- Node Worker 必须使用 `main.js`，并导出可调用的 `handler`
- Worker 的 `main` 文件中必须包含 `handler` 方法，否则 Worker 无法被调用
- 代码要按文档中的 `ctx.request`、`ctx.event` 结构读取输入
- 代码输出必须符合文档中的 stdout JSON 契约，至少返回合法的 `body`，并正确使用 `status`、`headers`、`file`
- 如果修改入口文件逻辑，不能脱离文档约定自定义一套输入输出协议
- 默认遵循单文件原则，优先在 `main.py` 或 `main.js` 中实现功能；只有在功能繁杂、代码量较多时，才考虑拆分多文件

## 路由规则

- `route` 必须以 `/` 开头
- 通配符仅支持结尾 `/*`
- 路由不明确时，不要自行补全复杂规则，先向用户确认

## 文件规则

- 只支持 Worker 根目录单层文件名
- `filename` 不能包含 `/`
- 如果涉及代码编写或逻辑修改，先用 `list_worker_files` 查看文件列表
- 拉取文件时，优先拉取入口文件 `main.py` 或 `main.js`
- 只有在本次开发需求、代码引用关系或本地 CLI 测试报错指向其他文件时，再继续用 `get_worker_file` 拉取相关文件
- 对图片、媒体等非文本类型文件，优先使用本地 mock 的方式完成测试；只有在测试不通过，或代码强依赖某个具体文件时，再考虑拉取原文件
- `upload_worker_file` 与 `update_worker_file` 都会覆盖已有内容

## 测试规则

- 代码改动完成后，必须先在本地环境通过 CLI 命令测试代码是否正常运行
- 必要时可以临时编写测试文件辅助验证，但最终上传时可以不上传测试文件
- 代码上传到线上 Callit 平台后，应通过 `curl` 测试对应 HTTP 接口
- 部署后测试需要确认接口可访问、状态码正确、返回内容正确
- 如果不清楚线上 `host`，必须先询问用户提供
- 如果用户明确允许跳过部署后测试，才可以不执行这一步

## 删除规则

- 删除前先确认文件存在
- 删除前先确认不是入口文件
- `main.py` 与 `main.js` 不能删除

## 安全行为

- 不要在未读取现状的情况下盲目覆盖用户文件，除非用户明确要求直接覆盖
- 搜索结果不唯一时，先让用户确认目标 Worker
- 不要声称某次创建、修改或删除已经成功，除非 MCP 返回成功结果
