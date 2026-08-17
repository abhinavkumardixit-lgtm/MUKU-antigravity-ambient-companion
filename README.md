# MUKU (Model-agnostic User-in-the-loop Kinetic Utility)

MUKU is an ambient, distributed desktop daemon designed to solve the Human-in-the-Loop (HITL) oversight bottleneck in autonomous AI developer agents[cite: 1]. It decouples agent execution telemetry and permission gating from the terminal and IDE into a lightweight, non-blocking visual overlay[cite: 1].

---

## Key Features

* **Asynchronous Request-Correlation Protocol:** Connects synchronous agent tool-calling loops with an event-driven PyQt6 overlay using a local FastAPI/asyncio WebSocket bridge[cite: 1].
* **Multi-Provider Cascade Routing:** Intercepts provider rate limits (`HTTP 429`) and transparently routes execution state across foundation models (Claude $\rightarrow$ Gemini $\rightarrow$ GPT) to prevent task drops[cite: 1].
* **Non-Blocking Ambient Daemon:** A frameless, transparent, always-on-top companion that presents single-click authorization cards without capturing window focus or disrupting active editor workflows[cite: 1].

---

## Project Structure

```text
antigravity-ambient-companion/
├── requirements.txt
├── docs/
│   └── architecture.md
└── src/
    ├── bridge_server.py
    ├── pet_overlay.py
    └── agent_hook.py
