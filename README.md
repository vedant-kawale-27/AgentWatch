
# AgentWatch

> Catch what your AI agent breaks — before it breaks production.

AgentWatch sits between your AI agent and the world. It watches every 
action, scores every reasoning step, blocks dangerous commands, and 
gives you a full replay of exactly what went wrong and why.

No more silent failures. No more "the agent said it succeeded."

![CI](https://github.com/sreerevanth/AgentWatch/actions/workflows/ci.yml/badge.svg)
![Python 3.12+](https://img.shields.io/badge/python-3.12+-blue.svg)
![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)

---

## The Problem

Your AI agent just deleted the wrong folder. Or worse — it succeeded, 
returned a confident response, and you found out three hours later when 
nothing worked.

76% of AI agent deployments fail within 90 days. Not because the models 
are bad. Because nobody can see what the agent is doing until it's 
already done something irreversible.

AgentWatch fixes that.

---

## What It Does

| | |
|---|---|
| 🛡️ **Safety Engine** | Blocks dangerous commands before they execute |
| 🧠 **Reasoning Auditor** | Scores every reasoning step in real time — not just the final output |
| 📊 **Live Dashboard** | Full execution trace of every action your agent takes |
| ⏪ **One-Click Rollback** | Git-backed filesystem checkpoints at every step |
| 💾 **Persistent Memory** | Episodic, semantic, and procedural memory across sessions |
| 💰 **Cost Tracker** | Per-session token budget tracking with live spend alerts |
| 🔔 **Alerting** | Slack and PagerDuty webhooks when confidence drops or actions are blocked |
| 📋 **Compliance Reports** | GDPR/HIPAA-ready audit exports with full RBAC governance |
| 🔌 **Works With Everything** | Claude Code, LangChain, AutoGPT, OpenClaw — drop-in, no rewrites |

---

## Quick Start

```bash
pip install agentwatch
docker compose up -d
```

Dashboard → http://localhost:3000
API → http://localhost:8000

---

## Supported Agents

AgentWatch wraps your existing agent. You don't rewrite anything.

### Claude Code

```bash
agentwatch watch "Build me a REST API for a todo app"
```

### LangChain

```python
from agentwatch.adapters.langchain import AgentWatchCallbackHandler

handler = AgentWatchCallbackHandler()
agent_executor = AgentExecutor(agent=..., callbacks=[handler])
```

### AutoGPT

```python
from agentwatch.adapters.autogpt import AutoGPTAdapter

adapter = AutoGPTAdapter(session_id="my-session")
await adapter.on_action(action)
```

### OpenClaw

```python
from agentwatch.adapters.openclaw import OpenClawAdapter

adapter = OpenClawAdapter(session_id="my-session")
await adapter.on_skill_execution(skill_name, payload)
```

---

## How It Works

Every agent action flows through AgentWatch before it executes:

```
Agent → AgentWatch → Safety Check → Reasoning Audit → Execute → Trace
                          ↓                ↓
                     Block if risky    Alert if confidence drops
```

Every event is normalized to the Universal Event Schema, stored in 
Postgres, streamed to the dashboard via WebSocket, and scored by the 
reasoning auditor in real time.

---

## Safety Engine

Blocks dangerous commands before they run:

```python
from agentwatch.core.safety import SafetyEngine

engine = SafetyEngine()
result = await engine.check_event(event)

if result.is_blocked:
    print(f"Blocked: {result.safety.reasons}")
    print(f"Risk: {result.safety.risk_level.value}")
```

Blocked by default: `rm -rf /`, `curl | bash`, disk formatting, 
credential exfiltration, and 40+ other critical patterns.

---

## Reasoning Auditor

The core feature nobody else has. Scores every reasoning step — not 
just the output — using an independent LLM-as-judge:

```python
from agentwatch.reasoning.auditor import ReasoningAuditor

auditor = ReasoningAuditor()
result = await auditor.score_step(reasoning_step)

print(result.confidence)         # 0.0 – 1.0
print(result.hallucination_risk) # low / medium / high
print(result.goal_drift)         # True if agent is off-task
```

When confidence drops below your threshold, AgentWatch fires an alert 
before the next action executes — not after it causes damage.

---

## Rollback

Every checkpoint is a full filesystem snapshot backed by git:

```bash
# Via CLI
agentwatch rollback <session-id> --to-step 12

# Via dashboard
# Select checkpoint → click Rollback
```

---

## Persistent Memory

Your agent remembers context across sessions:

```python
from agentwatch.memory.engine import MemoryEngine, MemoryType

memory = MemoryEngine()

await memory.store(
    agent_id="my-agent",
    content="Production DB is at 10.0.0.1:5432",
    memory_type=MemoryType.SEMANTIC,
)

results = await memory.retrieve(
    agent_id="my-agent",
    query="database connection"
)
```

---

## REST API

```
GET  /api/v1/sessions
GET  /api/v1/sessions/{id}/replay
GET  /api/v1/sessions/{id}/confidence
GET  /api/v1/sessions/{id}/checkpoints
POST /api/v1/sessions/{id}/rollback
GET  /api/v1/safety/blocked
GET  /api/v1/dashboard/summary
WS   /ws/events
```

Full API docs at http://localhost:8000/docs

---

## CLI

```bash
agentwatch watch "<prompt>"        # Watch a Claude Code session
agentwatch serve                   # Start the API server
agentwatch replay <session.json>   # Replay any session
agentwatch sessions                # List all sessions
agentwatch safety "<command>"      # Risk-score any command
```

---

## Stack

- **Backend** — FastAPI, PostgreSQL, Redis, Celery
- **Frontend** — Next.js, Tailwind, Recharts, WebSockets
- **Infra** — Docker Compose, GitHub Actions CI
- **Observability** — OpenTelemetry compatible

---

## What AgentWatch Is Not

- Not an agent framework — it wraps agents, it doesn't replace them
- Not perfect reliability — it reduces risk, not eliminates it
- Not AGI safety research — this is production reliability tooling

---

## Verification

```
✅ 47/47 tests passing
✅ docker compose up — zero errors
✅ API live at localhost:8000
✅ Dashboard live at localhost:3000
✅ Claude Code, LangChain, AutoGPT, OpenClaw adapters confirmed
```

---

## License

Apache 2.0 — see LICENSE
```
