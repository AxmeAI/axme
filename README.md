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

## Quick Start

```bash
# Install the CLI
curl -fsSL https://raw.githubusercontent.com/AxmeAI/axme-cli/main/install.sh | sh

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

---

## Intent Lifecycle

```
CREATED → SUBMITTED → DELIVERED → ACKNOWLEDGED → IN_PROGRESS → WAITING → COMPLETED
                                                                       ↘ FAILED
                                                                       ↘ CANCELLED
                                                                       ↘ TIMED_OUT
```

---

## Repository Map

| Repository | Description |
|---|---|
| **[axme](https://github.com/AxmeAI/axme)** | This repo — project overview and entry point |
| **[axme-docs](https://github.com/AxmeAI/axme-docs)** | API reference, integration guides, protocol docs |
| **[axme-examples](https://github.com/AxmeAI/axme-examples)** | Runnable examples across all SDKs |
| **[axme-cli](https://github.com/AxmeAI/axme-cli)** | CLI — manage intents, agents, scenarios, tasks |
| **[axp-spec](https://github.com/AxmeAI/axp-spec)** | AXP protocol specification |
| **[axme-conformance](https://github.com/AxmeAI/axme-conformance)** | Conformance test suite for spec-runtime-SDK parity |

---

## SDKs

All SDKs implement the same AXP protocol surface. All are currently at **v0.1.2 (Alpha)**.

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
```

---

## AXP — the Intent Protocol

AXP is the open protocol behind AXME. It defines the intent envelope, lifecycle states, delivery semantics, and contract model. AXP can be implemented independently of AXME Cloud — the spec, SDKs, and conformance suite are all public.

Protocol spec: [axp-spec](https://github.com/AxmeAI/axp-spec)

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
- **Protocol spec**: [axp-spec](https://github.com/AxmeAI/axp-spec)
- **Contact**: [hello@axme.ai](mailto:hello@axme.ai)
- **Security**: [SECURITY.md](SECURITY.md)

---

## Contributing

This repository is the entry point — not the implementation. To contribute:

- **Protocol / schemas** → [axp-spec](https://github.com/AxmeAI/axp-spec)
- **Documentation** → [axme-docs](https://github.com/AxmeAI/axme-docs)
- **SDK improvements** → respective SDK repository
- **Examples** → [axme-examples](https://github.com/AxmeAI/axme-examples)
- **Conformance checks** → [axme-conformance](https://github.com/AxmeAI/axme-conformance)

See [CONTRIBUTING.md](CONTRIBUTING.md) · [SECURITY.md](SECURITY.md) · [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
