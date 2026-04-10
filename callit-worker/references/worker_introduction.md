# Worker 文档
你可以把 Worker 理解为一个轻量级的函数计算环境，每个 Worker 都有独立的运行时和文件空间，通过 HTTP 触发执行。  

在 Worker 中，你可以使用 Python 或 Node.js 编写自定义代码来处理 HTTP 请求，实现动态响应、数据处理、文件操作等功能。 

除官方库之外，你还可以安装第三方依赖来扩展 Worker 的能力，例如使用 `requests` 库在 Python Worker 中发起 HTTP 请求，或在 Node Worker 中使用 `lodash` 进行数据处理。具体参考 [依赖库来源](#依赖库来源) 一节。

## 工作原理

每个 Worker 对应一个独立目录，目录路径通常为：

`{DATA_DIR}/workers/{worker_id}`

当 HTTP 请求命中某个 Worker 路由后，系统会：

1. 读取并解析请求内容。
2. 组装 `WorkerInput` 上下文。
3. 将 `WorkerInput` 以 JSON 形式写入脚本进程的 `stdin`。
4. 执行 Worker 主文件。
5. 读取脚本的 `stdout` / `stderr`。
6. 将 `stdout` 中的结构化结果解析为 HTTP 响应。
7. 当接收到文件上传时，系统会把文件暂存到服务器的运行时数据目录，并把文件信息（如路径、大小、类型）传入 `request.body[filename]` 中。沙箱内对应的可见路径为 `/tmp/upload`。

Worker 中的脚本仅允许读写 `/tmp` 文件夹，其他文件皆为**只读**。

另外，平台内置了 `kv` 能力，你可以使用它来存储、读取持久化的字符串数据。  

## 入口文件要求

Worker 目录中必须包含主代码文件，文件名由 runtime 决定：

- Python Worker：必须包含 `main.py`
- Node Worker：必须包含 `main.js`

如果主文件缺失，Worker 执行会直接失败。

在 `main.xx` 入口文件中必须提供 `handler` 方法，用于接收执行上下文并返回响应结果。

如果脚本通过 `result.file` 返回文件路径：

- 相对路径会按 Worker 根目录解析
- 以 `/tmp/` 开头的路径会按运行时数据目录解析

### Python

`main.py` 必须定义：

```python
def handler(ctx):
    ...
```

如果未定义可调用的 `handler(ctx)`，运行时会报错：

Python Worker 还可以直接使用平台注入的 `kv`：

```python
import json
from callit import kv

def handler(ctx):
    kv_client = kv.new_client("group1")
    kv_client.set("session", json.dumps({"id": "1234"}), 300)
    value = kv_client.get("session")
    return {
        "status": 200,
        "body": value
    }
```

### Node

`main.js` 必须通过 CommonJS 导出 `handler`：

```javascript
function handler(ctx) {
  ...
}

module.exports = handler;
```

或：

```javascript
exports.handler = function (ctx) {
  ...
}
```

如果没有导出可调用的 `handler`，运行时会报错：

Node Worker 也可以直接使用平台注入的 `kv`：

```javascript
const { kv } = require("callit");

async function handler(ctx) {
  const kvClient = kv.newClient("group1");
  await kvClient.set("session", JSON.stringify({ id: "1234" }), 300);
  const value = await kvClient.get("session");

  return {
    status: 200,
    body: value
  };
}

module.exports = handler;
```

## SDK 能力

Worker 运行时内置了 `kv` 和 `db` 两类 SDK 能力。

- `kv` 用于字符串键值存储
- `db` 用于访问 Worker 可用的共享数据库

详细的调用方式、参数规则、返回值结构和代码示例，请参考：

- [Worker SDK 文档](./worker_sdk.md)

## 模板示例

### Python

```python
def handler(ctx):
    request = ctx.get("request", {})

    return {
        "status": 200,
        "body": {
            "message": "Hello, Callit!",
            "request": request
        },
        "headers": {
            "Content-Type": "application/json"
        }
    }
```

### Node

```javascript
function handler(ctx) {
  const { request } = ctx;

  return {
    status: 200,
    body: {
      message: "Hello, Callit!",
      request,
    },
    headers: {
      "Content-Type": "application/json"
    }
  };
}

module.exports = handler
```

## context 结构

Worker 接收到的上下文模型为 `WorkerInput`：

```json
{
  "request": {},
  "event": {}
}
```

其中包含两部分：

- `request`：当前 HTTP 请求信息
- `event`：当前 Worker 执行事件信息

### request 字段说明

#### `request.method`

- 类型：`string`
- 来源：当前 HTTP 请求方法，如 `GET`、`POST`

#### `request.uri`

- 类型：`string`
- 来源：根据 Worker 路由和当前请求路径计算出的路由后缀
- 说明：
  - 如果 Worker 路由是 `/hello/*`
  - 实际请求路径是 `/hello/user/profile`
  - 则 `uri` 为 `/user/profile`
  - 如果没有额外后缀，则为 `/`

#### `request.url`

- 类型：`string`
- 来源：当前请求的完整 URL
- 说明：由请求的 scheme、host、path、query 重新拼装得到

#### `request.params`

- 类型：`map[string]string`
- 来源：URL query 参数
- 说明：
  - 来自 `?a=1&b=2`
  - 如果同名参数重复出现，保留最后一个值

#### `request.headers`

- 类型：`map[string]string`
- 来源：当前 HTTP 请求头
- 说明：
  - 所有请求头都会被传入
  - 同名 header 的多个值会使用逗号拼接成一个字符串

#### `request.body`

- 类型：`any`
- 来源：根据 `Content-Type` 对请求体做结构化解析后的结果
- 说明：
  - `application/json`：解析为 JSON 对象或数组
  - `application/x-www-form-urlencoded`：解析为键值对象
  - `multipart/form-data`：解析为表单字段和文件元信息
  - 其他类型：默认返回空对象 `{}`

#### `request.body_str`

- 类型：`string`
- 来源：原始 HTTP 请求体
- 说明：
  - 非 `multipart/form-data` 请求时，保留原始 body 字符串
  - `multipart/form-data` 请求时固定为空字符串
  - 这样做是为了避免把过大的上传内容直接塞进上下文

### event 字段说明

#### `event.request_id`

- 类型：`string`
- 来源：本次请求生成的唯一请求 ID
- 用途：可用于日志追踪、问题排查

#### `event.runtime`

- 类型：`string`
- 来源：当前 Worker 的运行时配置
- 可能值：`python` 或 `node`

#### `event.worker_id`

- 类型：`string`
- 来源：当前被执行 Worker 的唯一标识

#### `event.route`

- 类型：`string`
- 来源：当前命中的 Worker 路由规则

## stdin(Context) 样例

下面是一个典型的 `stdin` 样例。系统会把这段 JSON 文本写入脚本标准输入：

```json
{
  "request": {
    "method": "POST",
    "uri": "/user/profile",
    "url": "http://127.0.0.1:3100/api/hello/user/profile?name=callit",
    "params": {
      "name": "callit"
    },
    "headers": {
      "Content-Type": "application/json",
      "User-Agent": "curl/8.5.0"
    },
    "body": {
      "message": "hello"
    },
    "body_str": "{\"message\":\"hello\"}"
  },
  "event": {
    "request_id": "7b0d9f0a-9e95-45fd-ae43-3df2f6d2d001",
    "runtime": "python",
    "worker_id": "2bcf9922-c7d9-4d2e-a15d-b83de4ece1c6",
    "route": "/api/hello/*"
  }
}
```

## stdout 约定与样例

Worker 最终必须通过 `stdout` 输出一个合法 JSON，用于表示 HTTP 响应。

支持字段如下：

- `status`：HTTP 状态码，可选，默认 `200`
- `headers`：响应头，可选
- `file`：Worker 目录中的相对文件路径，可选
- `body`：响应体，必填

### stdout 样例

```json
{
  "status": 200,
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {
    "message": "ok"
  }
}
```

## 上传文件

当请求是 `multipart/form-data` 时，Worker 可以在 `request.body` 中读取文本字段和上传文件信息。

上传文件对象包含：

- `filename`：原始文件名
- `content_type`：文件类型
- `size`：文件大小（字节）
- `path`：文件在沙箱中的可访问路径

其中 `path` 通常位于 `/tmp/upload/...` 下，可以直接读取。

### Python 上传文件示例

```python
def handler(ctx):
    request = ctx.get("request", {})
    body = request.get("body", {})
    file_info = body.get("file")
    if not file_info:
        return {"status": 400, "body": {"error": "missing file"}}

    with open(file_info["path"], "r", encoding="utf-8") as f:
        content = f.read()

    return {
        "status": 200,
        "body": {
            "filename": file_info["filename"],
            "content": content
        }
    }
```

### Node 上传文件示例

```javascript
const fs = require("fs");

function handler(ctx) {
  const { request } = ctx;
  const fileInfo = request.body?.file;
  if (!fileInfo) {
    return {
      status: 400,
      body: { error: "missing file" }
    };
  }

  const content = fs.readFileSync(fileInfo.path, "utf8");
  return {
    status: 200,
    body: {
      filename: fileInfo.filename,
      content,
    }
  };
}

module.exports = handler;
```

## 下载文件

如果希望返回文件下载，可以通过 `file` 字段指定 Worker 目录中的相对路径，或 `/tmp/` 下的临时文件路径。

### Python 文件下载示例

```python
def handler(ctx):
    path = "/tmp/output.txt"
    with open(path, "w", encoding="utf-8") as f:
        f.write("hello from callit\n")
    return {
        "status": 200,
        "file": path
    }
```

### Node 文件下载示例

```javascript
const fs = require("fs");

function handler(ctx) {
  const path = "/tmp/output.txt";
  fs.writeFileSync(path, "hello from callit\n", "utf8");
  return {
    status: 200,
    file: path,
  };
}

module.exports = handler;
```

## 返回 HTML 页面

Worker 可以直接返回 HTML 字符串，并通过 `headers` 指定 `Content-Type`。

### Python HTML 示例

```python
def handler(ctx):
    return {
        "status": 200,
        "headers": {
            "Content-Type": "text/html; charset=utf-8"
        },
        "body": "<h1>Hello Callit</h1>"
    }
```

### Node HTML 示例

```javascript
function handler(ctx) {
  return {
    status: 200,
    headers: {
      "Content-Type": "text/html; charset=utf-8",
    },
    body: "<h1>Hello Callit</h1>",
  };
}

module.exports = handler;
```

## 依赖库来源

Worker 可以使用平台内置 SDK，也可以安装第三方依赖。

当前依赖来源通常包括：

- 平台运行时预置依赖
- Worker 自身安装的依赖

如果依赖安装成功但运行时报找不到包，优先检查 runtime 是否一致。

