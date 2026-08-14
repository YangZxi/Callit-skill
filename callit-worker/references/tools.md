# Callit MCP Tools

## `create_worker`

作用：

- 创建新的 Worker

关键参数：

- `name`
- `description` 可选
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
- `description` 可选
- `route`
- `timeout_ms`
- `enabled` 可选

限制：

- 不允许传 `runtime`

## `search_workers`

作用：

- 按名称模糊搜索 Worker

关键参数：

- `keyword`：可选关键词

返回重点：

- 匹配到的 Worker 列表，包含 `id`、`name`、`description`、`runtime`、`route`、`timeout_ms`、`enabled`

## `list_worker_files`

作用：

- 递归列出指定 Worker 代码目录中的文件和目录

关键参数：

- `worker_id`

返回重点：

- 相对路径数组；目录路径以 `/` 结尾

## `get_worker_file`

作用：

- 读取指定 Worker 文件内容

关键参数：

- `worker_id`
- `filename`

限制：

- `filename` 必须是文件相对路径，不能以 `/` 结尾

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

路径规则：

- `filename` 以 `/` 结尾时创建目录并忽略 `content`
- 文件路径遵循 Linux 命名规则；系统会自动移除路径开头的 `/` 和各级名称首尾空格
- 创建嵌套文件或目录时会自动创建缺少的父目录
- Worker 根目录计为第 1 层，服务端默认最多允许 3 层目录

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
- `filename` 以 `/` 结尾时创建目录并忽略 `content`
- 创建嵌套文件或目录时会自动创建缺少的父目录

## `delete_worker_file`

作用：

- 删除 Worker 中的指定文件或目录

关键参数：

- `worker_id`
- `filename`

限制：

- 不能删除 `main.py` 或 `main.js`
- 目录路径以 `/` 结尾
- 删除目录会递归删除其中全部文件和子目录，调用前必须先向用户明确说明并取得确认
