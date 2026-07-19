# OpenCode Server operations

Tested against OpenCode server **v1.18.3**.

## preflightCheck

```bash
curl -s http://{host}:{port}/global/health
# Expect: {"healthy":true,"version":"1.18.3"}
```

## dispatchBrief (sync)

```bash
# Step 1: Create session
SESSION=$(curl -s -X POST "http://{host}:{port}/session?directory=/path/to/repo" \
  -H "Content-Type: application/json" \
  -d '{"title": "{specname}-{taskname}-opsx-implementer"}')
SESSION_ID=$(echo "$SESSION" | jq -r '.id')

# Step 2: Send brief (blocks until complete)
RESPONSE=$(curl -s -X POST "http://{host}:{port}/session/$SESSION_ID/message" \
  -H "Content-Type: application/json" \
  -d '{"parts": [{"type": "text", "text": "'"$(cat brief.md)"'"}]}')

# Step 3: Extract text response
echo "$RESPONSE" | jq -r '.parts[] | select(.type == "text") | .text'
```

## dispatchBrief (async)

```bash
# Step 1: Create session (same as sync)
SESSION=$(curl -s -X POST "http://{host}:{port}/session?directory=/path/to/repo" \
  -H "Content-Type: application/json" \
  -d '{"title": "{specname}-{taskname}-opsx-implementer"}')
SESSION_ID=$(echo "$SESSION" | jq -r '.id')

# Step 2: Fire and forget — returns 204
curl -s -X POST "http://{host}:{port}/session/$SESSION_ID/prompt_async" \
  -H "Content-Type: application/json" \
  -d '{"parts": [{"type": "text", "text": "'"$(cat brief.md)"'"}]}' \
  -o /dev/null
```

## extractSessionId

```bash
SESSION_ID=$(echo "$SESSION" | jq -r '.id')
```

`$SESSION` is the full response from `POST /session`.

## readReport

Read the final assistant message:

```bash
curl -s "http://{host}:{port}/session/{session_id}/message" \
  | jq -r '.[-1].parts[] | select(.type == "text") | .text'
```

Read all messages with jq filtering:

```bash
curl -s "http://{host}:{port}/session/{session_id}/message" \
  | jq -r '.[] | "\(.info.role): \(.parts[] | select(.type == "text") | .text)"'
```

## resumeSession

```bash
curl -s -X POST "http://{host}:{port}/session/{session_id}/message" \
  -H "Content-Type: application/json" \
  -d '{"parts": [{"type": "text", "text": "continue with changes..."}]}'
```

## abortSession

```bash
curl -s -X POST "http://{host}:{port}/session/{session_id}/abort"
# Returns: true
```

## Agent and model override

```bash
curl -s -X POST "http://{host}:{port}/session/{session_id}/message" \
  -H "Content-Type: application/json" \
  -d '{
    "agent": "build",
    "model": {"providerID": "anthropic", "modelID": "claude-3-5-sonnet-20241022"},
    "parts": [{"type": "text", "text": "..."}]
  }'
```

## Sync vs async

| 场景 | 模式 | 理由 |
|------|------|------|
| 单个 task group | 同步 | 响应即完成，最简单 |
| 并行多个 task groups | 异步 | 非阻塞，同时发起多个 |
| 简单 smoke test | 同步 | 立即得到结果 |

Orchestrator 自行决定何时检查异步结果。
