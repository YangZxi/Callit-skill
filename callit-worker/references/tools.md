# Callit MCP Tools

## `search_workers`

作用：

- 按名称模糊搜索 Worker

关键参数：

- `keyword`：可选关键词

返回重点：

- 匹配到的 Worker 列表，包含 `id`、`name`、`runtime`、`route`、`timeout_ms`、`enabled`

## `list_worker_files`

作用：

- 列出指定 Worker 根目录下的文件名

关键参数：

- `worker_id`

返回重点：

- 文件名数组

## `get_worker_file`

作用：

- 读取指定 Worker 文件内容

关键参数：

- `worker_id`
- `filename`

返回重点：

- 文件内容
- 对图片类文件，可能返回预览信息而不是纯文本

## `upload_worker_file`

作用：

- 新建文件，或以“上传”语义覆盖已有文件

关键参数：

- `worker_id`
- `filename`
- `content`

副作用：

- 同名文件会被直接覆盖

## `update_worker_file`

作用：

- 更新指定 Worker 文件内容

关键参数：

- `worker_id`
- `filename`
- `content`

副作用：

- 会直接覆盖已有内容
- 文件不存在时也会创建

## `delete_worker_file`

作用：

- 删除 Worker 根目录下的指定文件

关键参数：

- `worker_id`
- `filename`

限制：

- 不能删除 `main.py` 或 `main.js`

## `create_worker`

作用：

- 创建新的 Worker

关键参数：

- `name`
- `runtime`
- `route`
- `timeout_ms`
- `enabled` 可选

副作用：

- 成功后会自动生成入口文件

## `update_worker`

作用：

- 更新 Worker 基础信息

关键参数：

- `id`
- `name`
- `route`
- `timeout_ms`
- `enabled` 可选

限制：

- 不允许传 `runtime`
