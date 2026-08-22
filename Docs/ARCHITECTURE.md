# Nemesis Architecture

Technical deep dive into how Nemesis works under the hood.

---

## System Overview

Nemesis is a multi-layer platform connecting AI assistants to pentesting environments:

![System Overview](../assets/doc/system-overview.png)

---

## MCP Protocol Flow

How commands flow from the AI CLI into the pentest container:

```
+----------------+   stdio/HTTP   +---------------------+   docker exec   +------------------+
|  AI CLI        | <------------> |  nemesis MCP server | --------------> |  pentest container|
| (claude/kimi/  |                |  (backend/mcp/)     |                 |  (nemesis-pentest |
|  qwen code)    |                |                     |                 |   or Exegol)      |
+----------------+                +----------+----------+                 +------------------+
                                             |
                                    FastAPI backend (api/v1)
                                             |
                          +------------------+------------------+
                          |                  |                  |
                    PostgreSQL          WebSocket         Reports (PDF)
                    (assessments,       (live updates)    via WeasyPrint
                     findings, creds)
```

1. The launcher (`nemesis.py`) authenticates to the backend, resolves the assessment
   workspace, and generates `.nemesis/mcp-config.json` pointing the AI CLI at the
   MCP server running from the backend venv.
2. The AI CLI calls MCP tools (`execute`, `scan`, `http_request`, `add_card`, ...).
3. Each tool call is dispatched through `backend/mcp/modules/` handlers, which run
   commands inside the pentest container via the Docker socket proxy.
4. Results stream back over MCP and are persisted through the FastAPI backend so the
   dashboard, timeline, and PDF reports stay in sync with what the agent did.

---

