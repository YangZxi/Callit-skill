# Callit Worker 工作流

## 创建新 Worker

按这个顺序执行：

1. 调用 `search_workers(keyword)`，确认没有同名或相近用途的 Worker，如果已存在疑似目标，先向用户确认是复用还是新建
2. 出具 Worker 名称、route 等基础信息，然后发给用户进行确认
   * route。路由默认需要以 `/<uuid>/` 开头（uuid 需要你生成），然后再接上 Worker 的功能相关的路径；如果用户提供了自定义的路径，则直接使用用户提供的路径。例如：`/<uuid>/cur_time`（你生成的），`/weather`（用户提供的）
3. 用刚刚的信息调用 `create_worker(...args)` 来进行新建
4. 如果要编写 Worker 代码，先读取 [worker_introduction.md](./worker_introduction.md)
5. 优先按单文件原则在 `main.py` 或 `main.js` 中实现功能；只有在逻辑明显复杂时再拆分多文件
6. 根据需要继续调用 `update_worker_file` 或 `upload_worker_file` 写入业务代码
7. 本地通过 CLI 命令测试代码能否正常运行，必要时可以临时编写测试文件辅助验证
8. 上传到线上后，通过 `curl` 测试对应 HTTP 接口
9. 如有必要，调用 `list_worker_files` 或 `get_worker_file` 进行确认

注意：

- `create_worker` 成功后会自动生成入口文件
- 不要在创建后立刻删除入口文件
- 如果不清楚线上 `host`，先向用户索取；如果用户允许，也可以跳过部署后测试

## 更新 Worker 基础信息

按这个顺序执行：

1. 调用 `search_workers(keyword)` 定位目标 Worker，如果你已经有确定的 Worker ID，可以直接跳过这一步
2. 若搜索结果不唯一，先让用户确认目标
3. 调用 `update_worker(id, name, route, timeout_ms, enabled)`
4. 返回更新后的 Worker 关键信息

注意：

- `update_worker` 不允许传 `runtime`

## 查看并修改 Worker 文件

按这个顺序执行：

1. 调用 `search_workers(keyword)` 定位目标 Worker，如果你已经有确定的 Worker ID，可以直接跳过这一步
2. `list_worker_files(worker_id)`
3. 优先拉取入口文件 `main.py` 或 `main.js`
4. 如果准备改动 Worker 入口代码或核心响应逻辑，先读取 [worker_introduction.md](./worker_introduction.md)
5. 根据本次开发需求、代码中的引用关系，以及本地 CLI 测试报错，再继续拉取相关文件
6. 对图片、媒体等非文本文件，优先使用本地 mock 的方式测试；只有在测试不通过，或代码强依赖某个具体文件时，再考虑拉取原文件
7. 优先按单文件原则在 `main.py` 或 `main.js` 中补充功能，只有在逻辑复杂时再拆分多文件
8. 根据需求选择：
   - 改已有文件：`update_worker_file`
   - 新增或覆盖上传文件：`upload_worker_file`
9. 本地通过 CLI 命令测试代码能否正常运行，必要时可以临时编写测试文件辅助验证
10. 上传到线上后，通过 `curl` 测试对应 HTTP 接口
11. 需要确认时，再次 `get_worker_file` 或 `list_worker_files`

默认不要跳过第 3 步，除非用户明确要求直接覆盖。
如果不清楚线上 `host`，先向用户索取；如果用户允许，也可以跳过部署后测试。

## 删除 Worker 文件

按这个顺序执行：

1. 调用 `search_workers(keyword)` 定位目标 Worker，如果你已经有确定的 Worker ID，可以直接跳过这一步
2. `list_worker_files(worker_id)`
3. 确认目标文件存在，且不是入口文件
4. `delete_worker_file(worker_id, filename)`
5. 返回删除后的文件列表或确认删除成功

注意：

- 入口文件 `main.py` 或 `main.js` 不能删除

## 何时需要停下来问用户

遇到这些情况不要自行猜测：

- 搜索结果有多个疑似 Worker
- 用户想修改的文件名不存在，但请求里又明确说是“更新现有文件”
- 用户描述的路由规则不清楚
- 用户要求修改 `runtime`
- `callit-mcp` 不可用或未配置
- 不清楚线上 Callit 平台的 `host`
- 用户是否允许跳过部署后测试
