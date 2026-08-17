# MUKU System Architecture & Communication Protocol

## Overview
MUKU provides an ambient, decoupled Human-in-the-Loop (HITL) mediation pipeline between an autonomous agent runtime and a composited OS overlay daemon.
```text
+-------------------+      HTTP POST /intercept      +----------------------+
|                   | ---------------------------->  |                      |
|  Agent Runtime    |                                |  FastAPI IPC Bridge  |
|  (Hook Function)  | <----------------------------  |  (127.0.0.1:8765)    |
+-------------------+       {"authorized": bool}     +----------------------+
                                                                ^
                                                                | WebSocket
                                                                v
                                                     +----------------------+
                                                     |                      |
                                                     |   PyQt6 Overlay UI   |
                                                     |   (Desktop Mascot)   |
                                                     +----------------------+

## 1. Asynchronous Request-Correlation Protocol
                                  Agent Shell                     IPC Bridge Server                     PyQt6 Overlay UI
                                         |                                   |                                     |
                                         |-- 1. POST /intercept ------------>|                                     |
                                         |      (tool name, params)          |-- 2. Create Future[Correlation]  |
                                         |                                   |-- 3. WS Broadcast (AUTH_REQ) ------>|
                                         |   [Agent Thread Suspended]        |                                     |-- 4. Display Card
                                         |                                   |-- 5. User Click (Allow/Deny)
                                         |                                   |<-- 6. WS Response (Status) ---------|
                                         |                                   |-- 7. Resolve Future[Correlation] |
                                         |<-- 8. Return HTTP 200 (auth) -----|                                     |
                                         |                                   |                                     |
                               [Execution Resumes]
### Sequence Breakdown
1. **Interception**: When a protected tool execution begins, `agent_hook.py` issues a synchronous blocking HTTP POST to `/intercept` containing `tool_name` and `tool_params`.
2. **Correlation ID Generation**: The IPC bridge generates a unique `UUIDv4` identifier ($C_{id}$) and registers an unresolved `asyncio.Future` in `pending_requests[C_id]`.
3. **WebSocket Event Dispatch**: The bridge broadcasts the transaction envelope to the PyQt6 daemon over `ws://127.0.0.1:8765/ws`.
4. **Overlay Render**: The desktop daemon parses the packet and renders an ephemeral approval card adjacent to the floating mascot.
5. **Resolution**: The user clicks `Allow` or `Deny`, returning the decision status.
6. **Future Resolution**: The bridge maps the correlation ID to the awaiting future and sets the result.
7. **Agent Unblocking**: The `/intercept` HTTP route returns `HTTP 200 OK` with `{"authorized": true/false}`, allowing the agent to proceed or safely abort.

---

## 2. Multi-Provider Cascade Router Architecture

To guarantee uninterrupted workflow completion during API rate limiting (HTTP 429), the cascade layer intercepts provider exceptions and shifts model execution sequentially:
Primary Provider (Claude 3.7) 
         │
    [HTTP 429 / Quota Error]
         ▼
Secondary Provider (Gemini 2.5 Pro)
         │
    [HTTP 429 / Quota Error]
         ▼
Tertiary Provider (GPT-4o)
* **Session Schema Normalization**: Standardizes tool schemas and message history across different vendor SDKs.
* **Telemetry Push**: Emits `STATE_CASCADE_FAILOVER` over the local WebSocket to update the ambient pet badge in real time.

---

## 3. Directory Layout & Module Responsibilities

* **`src/bridge_server.py`**: Localhost FastAPI + `asyncio` broker maintaining the correlation state table and WebSocket connections.
* **`src/pet_overlay.py`**: Frameless, transparent PyQt6 GUI using `Qt.WindowType.WindowStaysOnTopHint` to render the mascot and authorization cards without capturing active editor focus.
* **`src/agent_hook.py`**: Python client hook facilitating synchronous-to-asynchronous translation for tool execution loops.
