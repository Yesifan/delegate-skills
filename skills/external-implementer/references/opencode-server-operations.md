# OpenCode Server operations

Tested against OpenCode server **v1.18.3**.

All paths are relative to `http://{host}:{port}`.

## Authentication

If `OPENCODE_SERVER_PASSWORD` is set, the server requires HTTP Basic Auth.
Include the following header on every request:

```
Authorization: Basic <base64>
```

Where `<base64>` = `base64("{username}:{password}")`, with username
defaulting to `opencode` (overridable via `OPENCODE_SERVER_USERNAME`).
The orchestrator should read these from the environment before making
any API calls.

All endpoints below require this header when auth is enabled.

## Connecting

When the user chooses OpenCode Server, determine the server's address:

1. **From user prompt**
2. **Auto-discover** from running processes:
   - Probe `http://127.0.0.1:4096/global/health`.
   - `ps aux | grep "opencode serve"` and parse `--port <N>`.
   - mDNS: if the server was started with `--mdns`, try `http://opencode.local:4096`.

3. **Ask user**:
   - If the server is not running, ask the user to start it with `opencode serve`.
   - If the server is running but not reachable, ask for host and port.

---

## preflightCheck

```
GET /global/health
```

Include `Authorization` header if auth is enabled. Returns `{ healthy: true, version }` on 200.

## dispatchBrief (sync)

Create a session and send the brief synchronously.

**Step 1 — Create session**

```
POST /session?directory=/path/to/repo
Body: {"title": "{specname}-{taskname}-opsx-implementer"}
```

Returns: `{ id: "ses_xxx", ... }`

**Step 2 — Send brief**

```
POST /session/:id/message
Body: {"parts": [{"type": "text", "text": "..."}]}
```

Blocks until the implementer finishes.

## dispatchBrief (async)

Create a session and dispatch without waiting.

**Step 1 — Create session** (same body as sync)

**Step 2 — Dispatch async**

```
POST /session/:id/prompt_async
Body: {"parts": [{"type": "text", "text": "..."}]}
```

Returns `204 No Content` immediately. Check results later via `readReport`.

## extractSessionId

Extract session ID from the `POST /session` response.

## readReport

Read session messages. Use this to get the implementer's final report.

```
GET /session/:id/message
Returns: {
  info: {
    id: "msg_xxx"
    role: "user" | "assistant";
    agent?: "plan" | "build"
    finish?: "stop" | "tool-calls";
    cost?: number;
    tokens?: { input: number; output: number; reasoning?: number };
  };
  parts: {
    id: string;
    type: "text" | "reasoning" | "step-start" | "step-finish" | "tool";
    text?: string;       // present when type == "text" or "reasoning"
    tool?: string;       // present when type == "tool"
  }[];
}[]
```

Messages are ordered oldest first.

## resumeSession

Continue an existing session. Context is preserved.

```
POST /session/:id/message
Body: {"parts": [{"type": "text", "text": "..."}]}
```

## abortSession

```
POST /session/:id/abort

Returns: bool
```

## listSessionPermissions

If the session gets stuck, you can check whether the permission request is stuck.

```
GET /api/session/:id/permission
Returns:{
  data:{
    id: "per_xxx" — use for replyPermission
    sessionID: "ses_xxx"
    action: "bash" | "edit" | ...
    resources: ["npm install"]
    save?: ["npm *"] — suggested rules to remember
    source?: {
      type: "tool";
      messageID: "msg_xxx"
      callID: tool call ID
    };
  }[]
};
```

## replyPermission

```
POST /permission/:id/reply
Body: {"reply": "once"|"always"|"reject", "message?": "..."}
```

## Agent and model override

Override agent and/or model per message. These fields can be added to any `POST /session/:id/message` body.

```
POST /session/:id/message
Body: {"agent": "build", "model": {"providerID": "...", "modelID": "..."}, "parts": [...]}
```
