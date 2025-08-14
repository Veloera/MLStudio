# VeloeraML API 文档

## 基本信息

- **版本**：1.0.0
- **Base URL**：`https://mlapi-beta.veloera.org/v1`

---

## 目录

- [模型列表](#模型列表-get-models)
- [聊天补全](#聊天补全-post-chatcompletions)
- [无状态文本生成](#无状态文本生成-post-generate)
- [有状态续写](#有状态续写-post-continue)
- [查询续写任务状态](#查询续写任务状态-get-continueid)
- [数据结构说明](#数据结构说明)

---

## 模型列表 (GET /models)

- **功能**：获取当前可用模型列表，格式兼容 OpenAI `/v1/models`，并额外包含 `description` 和 `series` 字段。
- **请求方式**：`GET /models`
- **响应示例**：

```json
{
  "object": "list",
  "data": [
    {
      "id": "lumeni-3",
      "object": "model",
      "created": 1692627200,
      "owned_by": "veloera-ml",
      "permission": [],
      "description": "快速的通用模型，专为速度敏感型任务设计",
      "series": "lumeni"
    }
  ]
}
```

---

## 聊天补全 (POST /chat/completions)

- **功能**：以聊天形式生成内容，兼容 OpenAI `/v1/chat/completions`。支持普通 JSON 返回和 SSE 流式输出（需设置 `stream=true`）。
- **请求方式**：`POST /chat/completions`
- **请求体**：

```json
{
  "model": "lumeni-3",
  "messages": [
    {"role": "system", "content": "你是助手"},
    {"role": "user", "content": "你好"}
  ],
  "stream": false
}
```

- **响应（JSON）**：

```json
{
  "id": "xxx",
  "object": "chat.completion",
  "created": 1692627200,
  "model": "lumeni-3",
  "choices": [
    {
      "index": 0,
      "message": {"role": "assistant", "content": "你好！"},
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 5,
    "total_tokens": 15
  }
}
```

- **响应（SSE 流式）**：

每个事件包含如下结构：

```json
{
  "event": "completion_chunk",
  "data": {
    "id": "xxx",
    "status": "in_progress",
    "response": {
      "content": "你好",
      "chunk": "你"
    }
  }
}
```

---

## 无状态文本生成 (POST /generate)

- **功能**：根据 system、上下文和 prompt 一次性生成结果，不记录会话状态。支持流式输出。
- **请求方式**：`POST /generate`
- **请求体**：

```json
{
  "system": "你是助手",
  "context": [
    {"role": "user", "content": "请介绍一下你自己"}
  ],
  "prompt": "你好",
  "stream": false,
  "model": "lumeni-3"
}
```

- **响应（JSON）**：

```json
{
  "success": true,
  "id": "xxx",
  "created_at": 1692627200,
  "model": "lumeni-3",
  "status": "completed",
  "response": {
    "content": "你好，我是 VeloeraML 的智能助手。"
  }
}
```

- **响应（SSE 流式）**：

```json
{
  "event": "response_chunk",
  "data": {
    "id": "xxx",
    "status": "in_progress",
    "response": {
      "content": "你好，我是",
      "chunk": "你好"
    }
  }
}
```

---

## 有状态续写 (POST /continue)

- **功能**：从之前的生成结果继续生成，可选传入 system。支持流式输出。
- **请求方式**：`POST /continue`
- **请求体**：

```json
{
  "previous_continue_id": "xxx",
  "system": "你是助手",
  "prompt": "请继续",
  "stream": false,
  "model": "lumeni-3"
}
```

- **响应（JSON）**：

```json
{
  "success": true,
  "id": "yyy",
  "created_at": 1692627201,
  "model": "lumeni-3",
  "status": "completed",
  "response": {
    "content": "好的，继续为您介绍..."
  }
}
```

- **响应（SSE 流式）**：

```json
{
  "event": "response_chunk",
  "data": {
    "id": "yyy",
    "status": "in_progress",
    "response": {
      "content": "好的，继续",
      "chunk": "好"
    }
  }
}
```

---

## 查询续写任务状态 (GET /continue/{id})

- **功能**：获取续写任务的最新状态和已生成结果。
- **请求方式**：`GET /continue/{id}`
- **路径参数**：
  - `id`：续写任务 ID（必填）

- **响应示例**：

```json
{
  "success": true,
  "id": "yyy",
  "created_at": 1692627201,
  "model": "lumeni-3",
  "status": "completed",
  "response": {
    "content": "好的，继续为您介绍..."
  }
}
```

---

## 数据结构说明


### ModelsList

- `object`: 返回对象类型，通常为 `list`。
  - 可能的值：`list`
- `data`: 模型数组，每项包含：
  - `id`: 模型唯一标识符。
    - 例如：`lumeni-3`
  - `object`: 类型标识，通常为 `model`。
    - 可能的值：`model`
  - `created`: 创建时间，Unix 时间戳（秒）。
    - 例如：`1692627200`
  - `owned_by`: 模型所有者。
    - 例如：`veloera-ml`
  - `permission`: 权限列表，通常为空数组或包含权限对象。
  - `description`: 模型描述信息。
    - 例如：`最新一代推理优化大语言模型，支持长上下文`
  - `series`: 模型系列。
    - 例如：`lumeni`

### ChatCompletionRequest

- `model`: 指定使用的模型 ID。
  - 例如：`lumeni-3`
- `messages`: 聊天消息数组，每条需包含：
  - `role`: 消息角色。
    - 可能的值：`system`（系统提示）、`user`（用户）、`assistant`（助手）
  - `content`: 消息内容。
- `stream`: 是否启用流式输出。
  - 可能的值：`true`（流式）、`false`（普通），默认 `false`

### ChatCompletionResponse

- `id`: 任务唯一标识符。
- `object`: 返回对象类型。
  - 可能的值：`chat.completion`
- `created`: 创建时间，Unix 时间戳。
- `model`: 使用的模型 ID。
- `choices`: 生成结果数组，每项包含：
  - `index`: 结果序号。
  - `message`: 生成的消息对象。
    - `role`: 消息角色（如 `assistant`）。
    - `content`: 生成内容。
  - `finish_reason`: 结束原因。
    - 可能的值：`stop`、`length` 等
- `usage`: token 使用情况。
  - `prompt_tokens`: 输入 token 数。
  - `completion_tokens`: 输出 token 数。
  - `total_tokens`: 总 token 数。

### SSEChatCompletionEvent

- `event`: 事件类型。
  - 可能的值：`completion_start`、`completion_chunk`、`completion_finish`
- `data`: 事件数据对象。
  - `id`: 任务 ID。
  - `status`: 当前状态。
    - 可能的值：`started`、`in_progress`、`completed`
  - `response`: 生成内容对象。
    - `content`: 当前内容。
    - `chunk`: 当前分片内容。

### GenerateRequest

- `system`: 系统提示信息。
- `context`: 上下文数组，每项包含：
  - `role`: 消息角色。
    - 可能的值：`user`、`assistant`
  - `content`: 消息内容。
- `prompt`: 当前输入提示。
- `stream`: 是否启用流式输出。
  - 可能的值：`true`、`false`，默认 `false`
- `model`: 指定使用的模型 ID。

### GenerateResponse

- `success`: 是否成功。
  - 可能的值：`true`、`false`
- `id`: 任务唯一标识符。
- `created_at`: 创建时间，Unix 时间戳。
- `model`: 使用的模型 ID。
- `status`: 任务状态。
  - 可能的值：`completed`、`failed`
- `response`: 生成内容对象。
  - `content`: 生成的文本内容。

### ContinueRequest

- `previous_continue_id`: 上一次续写任务 ID（可选）。
- `system`: 系统提示信息（可选）。
- `prompt`: 当前输入提示。
- `stream`: 是否启用流式输出。
  - 可能的值：`true`、`false`
- `model`: 指定使用的模型 ID。

### ContinueResponse

- `success`: 是否成功。
  - 可能的值：`true`、`false`
- `id`: 任务唯一标识符。
- `created_at`: 创建时间，Unix 时间戳。
- `model`: 使用的模型 ID。
- `status`: 任务状态。
- `response`: 生成内容对象。
  - `content`: 生成的文本内容。

### SSEGenerateEvents / SSEContinueEvents

- `event`: 事件类型。
  - 可能的值：`generate_start`、`response_chunk`、`generate_finish`（文本生成）；`continue_start`、`response_chunk`、`continue_finish`（续写）
- `data`: 事件数据对象。
  - `id`: 任务 ID。
  - `status`: 当前状态。
    - 可能的值：`started`、`in_progress`、`completed`
  - `response`: 生成内容对象。
    - `content`: 当前内容。
    - `chunk`: 当前分片内容。
