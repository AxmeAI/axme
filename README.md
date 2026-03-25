# AXME

**Durable execution where agents, services, and humans coordinate as equals.**

Submit once, track lifecycle, complete later. Replace polling, webhook glue, and Temporal complexity with one protocol for all your workflows — AI-driven or not.

[![Alpha](https://img.shields.io/badge/status-alpha-orange)](https://cloud.axme.ai/alpha/cli)

> **Alpha** — install CLI, log in, run your first example in under 5 minutes.
> [Quick Start](https://cloud.axme.ai/alpha/cli) · [cloud.axme.ai](https://cloud.axme.ai) · [hello@axme.ai](mailto:hello@axme.ai)

---

## What Is AXME?

AXME is a coordination layer for operations that take minutes, hours, or days to complete. An **intent** carries a payload, a delivery target, and a lifecycle policy. The platform drives it to a terminal state through retries, timeouts, reminders, and human approval steps — without external orchestration or polling.

AXME is not async RPC. Not a simplified Temporal. Not an agent framework. Not an MCP replacement. It is a protocol-based runtime where AI agents, backend services, and human operators participate as equal actors.

---

## Before and After

**Without AXME** - polling, webhooks, Redis, and glue code:

```python
import requests, time, redis

# 1. Submit the job
resp = requests.post("https://api.vendor.com/generate", json=payload)
job_id = resp.json()["job_id"]

# 2. Poll until done (or timeout after 10 min)
for _ in range(120):
    status = requests.get(f"https://api.vendor.com/jobs/{job_id}").json()
    if status["state"] in ("completed", "failed"):
        break
    time.sleep(5)

# 3. Webhook handler for async callbacks
@app.post("/webhooks/vendor")
def handle_webhook(req):
    r = redis.Redis()
    r.set(f"job:{req.json['job_id']}", req.json["result"])
    return {"ok": True}

# 4. Fetch result from Redis
result = redis.Redis().get(f"job:{job_id}")
```

**With AXME** - submit once, track lifecycle, done:

```python
from axme import AxmeClient, AxmeClientConfig

client = AxmeClient(AxmeClientConfig(api_key="axme_sa_..."))
intent = client.send_intent("agent://myorg/prod/generator", payload)
result = client.wait_for(intent["id"])  # retries, timeouts, delivery - all handled
```

No polling. No webhooks. No Redis. No glue code. The platform handles retries, timeouts, delivery guarantees, and human approval steps for you.

---

## Quick Start

```bash
# Install the CLI
curl -fsSL https://raw.githubusercontent.com/AxmeAI/axme-cli/main/install.sh | sh
# Open a new terminal, or run the "source" command shown by the installer

# Authenticate
axme login

# Run a built-in example: human approval via CLI
axme examples run human/cli
```

The `human/cli` example deploys a readiness-checker agent and pauses for human approval. You approve or reject directly from the terminal with `axme tasks approve <task_id>`.

---

## Connect Your Agents

An agent listens for intents, processes them, and resumes with a result:

```python
from axme import AxmeClient, AxmeClientConfig

client = AxmeClient(AxmeClientConfig(api_key="axme_sa_..."))

for delivery in client.listen("agent://myorg/myworkspace/my-agent"):
    intent = client.get_intent(delivery["intent_id"])
    result = process(intent["payload"])
    client.resume_intent(delivery["intent_id"], result)
```

---

## ScenarioBundle

A ScenarioBundle is a JSON file that declares agents, human roles, workflow steps, and an intent — everything needed to run a coordination scenario:

```json
{
  "scenario_id": "human.cli.v1",
  "agents": [
    {
      "role": "checker",
      "address": "deploy-readiness-checker",
      "delivery_mode": "stream",
      "create_if_missing": true
    }
  ],
  "humans": [
    { "role": "operator", "display_name": "Operations Team" }
  ],
  "workflow": {
    "steps": [
      { "step_id": "readiness_check", "assigned_to": "checker" },
      { "step_id": "ops_approval", "assigned_to": "operator", "requires_approval": true }
    ]
  },
  "intent": {
    "type": "intent.deployment.approval.v1",
    "payload": { "service": "api-gateway", "version": "3.2.1" }
  }
}
```

Apply it:

```bash
axme scenarios apply scenario.json --watch
```

---

## MCP — AI Assistant Integration

AXME exposes a full MCP (Model Context Protocol) server at `mcp.cloud.axme.ai`. AI assistants (Claude, ChatGPT, Gemini) can manage the entire platform through 48 tools — the same operations available in the CLI.

```
POST https://mcp.cloud.axme.ai/mcp
Authorization: Bearer <account_session_token>

{"jsonrpc": "2.0", "id": 1, "method": "tools/call",
 "params": {"name": "axme.intents_send", "arguments": {
   "to_agent": "agent://myorg/production/my-agent",
   "intent_type": "task.process.v1",
   "payload": {"data": "..."}
 }}}
```

Available tool groups: status, agents, intents, tasks, organizations, workspaces, members, quota, scenarios, sessions. See [connector setup guides](https://github.com/AxmeAI/axme-docs/tree/main/docs/connectors) for Claude, ChatGPT, and Gemini integration.

---

## Delivery Bindings

How intents reach agents and services:

| Binding | Transport | Use Case |
|---|---|---|
| `stream` | SSE (server-sent events) | Real-time agent listeners |
| `poll` | GET polling | Serverless / cron-based consumers |
| `http` | Webhook POST | Backend services with an HTTP endpoint |
| `inbox` | Human inbox | Human-in-the-loop tasks (approve, reject, respond) |
| `internal` | Platform-internal | Built-in platform steps (reminders, escalations) |

---

## Human-in-the-Loop

Three paths for human participation:

| Path | How It Works |
|---|---|
| **CLI** | `axme tasks list` → `axme tasks approve <task_id>` |
| **Email** | Magic link sent to the assigned human; click to approve/reject |
| **Form** | Custom form submitted via API or embedded UI |

Human steps pause the intent lifecycle. The platform handles reminders and timeouts automatically.

### Human Task Types

| Type | Purpose |
|---|---|
| `approval` | Binary yes/no decision gate (e.g., deploy go/no-go) |
| `form` | Structured data collection with custom fields |
| `review` | Content or artifact review with comments and verdict |
| `override` | Manual override of an automated decision or threshold |
| `confirmation` | Acknowledge receipt or verify a fact before proceeding |
| `assignment` | Route a work item to a specific person or team |
| `clarification` | Request missing information needed to continue |
| `manual_action` | Perform a physical or out-of-band action (e.g., flip a switch, sign a document) |

### Internal Runtime Steps

These steps run inside the platform - no agent or human action needed:

| Step | What It Does |
|---|---|
| `human_approval` | Pauses the intent and waits for a human decision before continuing |
| `timeout` | Fails or escalates the intent if a step exceeds its deadline |
| `reminder` | Sends a nudge to the assigned human after a configurable delay |
| `delay` | Pauses execution for a fixed duration before advancing to the next step |
| `escalation` | Re-routes a stalled task to a higher-priority handler or team |
| `notification` | Fires an event or message (email, Slack, webhook) without blocking the flow |

---

## Intent Lifecycle

```
CREATED → SUBMITTED → DELIVERED → ACKNOWLEDGED → IN_PROGRESS → WAITING → COMPLETED
                                                                       ↘ FAILED
                                                                       ↘ CANCELLED
                                                                       ↘ TIMED_OUT
```

---

<details>
<summary><h2>Repository Map</h2></summary>

| Repository | Description |
|---|---|
| **[axme](https://github.com/AxmeAI/axme)** | This repo - project overview and entry point |
| **[axme-docs](https://github.com/AxmeAI/axme-docs)** | API reference, integration guides, MCP connector setup |
| **[axme-examples](https://github.com/AxmeAI/axme-examples)** | Runnable examples across all SDKs |
| **[axme-cli](https://github.com/AxmeAI/axme-cli)** | CLI - manage intents, agents, scenarios, tasks |
| **[axme-spec](https://github.com/AxmeAI/axme-spec)** | AXP protocol specification |
| **[axme-conformance](https://github.com/AxmeAI/axme-conformance)** | Conformance test suite for spec-runtime-SDK parity |

</details>

---

## SDKs

All SDKs implement the same AXP protocol surface. All are currently at **v0.1.2 (Alpha)**.

<details>
<summary>SDK install commands (Python, TypeScript, Go, Java, .NET)</summary>

| SDK | Package | Install |
|---|---|---|
| **[Python](https://github.com/AxmeAI/axme-sdk-python)** | `axme` | `pip install axme` |
| **[TypeScript](https://github.com/AxmeAI/axme-sdk-typescript)** | `@axme/axme` | `npm install @axme/axme` |
| **[Go](https://github.com/AxmeAI/axme-sdk-go)** | `github.com/AxmeAI/axme-sdk-go/axme` | `go get github.com/AxmeAI/axme-sdk-go@latest` |
| **[Java](https://github.com/AxmeAI/axme-sdk-java)** | `ai.axme:axme-sdk` | Maven Central |
| **[.NET](https://github.com/AxmeAI/axme-sdk-dotnet)** | `Axme.Sdk` | `dotnet add package Axme.Sdk` |

### CLI

```bash
curl -fsSL https://raw.githubusercontent.com/AxmeAI/axme-cli/main/install.sh | sh
# Open a new terminal, or run the "source" command shown by the installer
```

</details>

---

## AXP — the Intent Protocol

AXP is the open protocol behind AXME. It defines the intent envelope, lifecycle states, delivery semantics, and contract model. AXP can be implemented independently of AXME Cloud — the spec, SDKs, and conformance suite are all public.

Protocol spec: [axme-spec](https://github.com/AxmeAI/axme-spec)

---

## Agent Addressing

Agents are addressed with a URI scheme:

```
agent://org/workspace/name
```

Example: `agent://acme/production/deploy-readiness-checker`

---

## Links

- **Cloud platform**: [cloud.axme.ai](https://cloud.axme.ai)
- **Quick Start**: [cloud.axme.ai/alpha/cli](https://cloud.axme.ai/alpha/cli)
- **API docs**: [axme-docs](https://github.com/AxmeAI/axme-docs)
- **Protocol spec**: [axme-spec](https://github.com/AxmeAI/axme-spec)
- **Contact**: [hello@axme.ai](mailto:hello@axme.ai)
- **Security**: [SECURITY.md](SECURITY.md)

---

<details>
<summary><h2>Contributing</h2></summary>

This repository is the entry point - not the implementation. To contribute:

- **Protocol / schemas** -> [axme-spec](https://github.com/AxmeAI/axme-spec)
- **Documentation** -> [axme-docs](https://github.com/AxmeAI/axme-docs)
- **SDK improvements** -> respective SDK repository
- **Examples** -> [axme-examples](https://github.com/AxmeAI/axme-examples)
- **Conformance checks** -> [axme-conformance](https://github.com/AxmeAI/axme-conformance)

See [CONTRIBUTING.md](CONTRIBUTING.md) - [SECURITY.md](SECURITY.md) - [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)

</details>
