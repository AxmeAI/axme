# AXME

**Durable execution infrastructure for long-running intents.**

AXME is a coordination layer for async operations that need delivery guarantees, retry logic, human-in-the-loop steps, and observable lifecycle — across services, agents, and time.

> **Alpha** · API surface is stabilizing. Not recommended for production workloads yet.  
> Alpha access: [cloud.axme.ai](https://cloud.axme.ai/alpha) · Questions: [hello@axme.ai](mailto:hello@axme.ai)

---

## What Is AXME?

AXME runs **intents** — durable actions that may take minutes, hours, or longer to complete.

An intent carries a payload, a delivery target, and a lifecycle policy. The platform guarantees it reaches a terminal state — through retries, timeouts, reminders, and human approval steps — without requiring external orchestration ticks.

```python
# Submit an intent. The platform drives it to completion.
intent = client.create_intent({
    "intent_type": "order.fulfillment.v1",
    "payload": {"order_id": "ord_123"},
    "owner_agent": "agent://fulfillment-service",
})

# Wait for resolution — retries and approvals happen server-side
result = client.wait_for(intent["intent_id"], terminal_states={"RESOLVED", "CANCELLED"})
print(result["status"])  # RESOLVED
```

---

## Get Started

1. **Get alpha access** → [cloud.axme.ai/alpha](https://cloud.axme.ai/alpha)
2. **Pick your SDK** and run the quickstart
3. **Explore examples** for more complex scenarios

### Quickstart (Python)

```bash
pip install axme
export AXME_API_KEY="axme_sa_..."
python -c "
from axme import AxmeClient, AxmeClientConfig
client = AxmeClient(AxmeClientConfig(api_key='${AXME_API_KEY}'))
print(client.health())
"
```

---

## AXP — the Intent Protocol

At the core of AXME is **AXP** — an open protocol for durable intent execution. AXP defines the envelope, lifecycle rules, and contract model. It can be implemented independently.

The open components (spec, SDKs, conformance, CLI) are all public. AXME Cloud is the managed runtime.

---

## Repository Map

### Start here

| Repository | What it is |
|---|---|
| **[axme](https://github.com/AxmeAI/axme)** | You are here — entry point and overview |
| **[axme-docs](https://github.com/AxmeAI/axme-docs)** | Full API reference, integration guides, protocol docs, diagrams |
| **[axme-spec](https://github.com/AxmeAI/axme-spec)** | Canonical protocol and public API schema contracts |
| **[axme-examples](https://github.com/AxmeAI/axme-examples)** | Runnable examples: cloud scenarios, protocol-only, advanced flows |

### SDKs

| Repository | Language | Status |
|---|---|---|
| **[axme-sdk-python](https://github.com/AxmeAI/axme-sdk-python)** | Python | GA |
| **[axme-sdk-typescript](https://github.com/AxmeAI/axme-sdk-typescript)** | TypeScript / Node.js | GA |
| **[axme-sdk-go](https://github.com/AxmeAI/axme-sdk-go)** | Go | Beta |
| **[axme-sdk-java](https://github.com/AxmeAI/axme-sdk-java)** | Java | Beta |
| **[axme-sdk-dotnet](https://github.com/AxmeAI/axme-sdk-dotnet)** | .NET / C# | Beta |

### Tooling

| Repository | What it is |
|---|---|
| **[axme-cli](https://github.com/AxmeAI/axme-cli)** | Go CLI — manage intents, contexts, agents, and service accounts from the terminal |
| **[axme-conformance](https://github.com/AxmeAI/axme-conformance)** | Contract test suite — validates spec-runtime-SDK parity |
| **[axme-reference-clients](https://github.com/AxmeAI/axme-reference-clients)** | Reference client implementations (planned) |

---

## Core Concepts

### Intent lifecycle

```
PENDING → PROCESSING → WAITING_* → DELIVERED → RESOLVED
                    ↘ CANCELLED / EXPIRED
```

An intent moves through states autonomously. `WAITING_HUMAN` pauses for an approval step. `WAITING_TIME` waits for a scheduled wakeup. The platform handles retries, timeouts, and reminders — no external polling required.

### Auth model

Every API call uses two optional credentials:

| Header | Purpose |
|---|---|
| `x-api-key` | Service/workspace API key — identifies the calling service |
| `Authorization: Bearer <token>` | Actor token — identifies the human or agent acting |

Most operations need only `x-api-key`. Multi-actor flows (approvals, delegated operations, org admin) require both.

### Key API families

| Family | What it covers |
|---|---|
| **Intents** | Create, get, list, cancel, resume, update controls |
| **Inbox / Approvals** | Human-in-the-loop steps, thread management, decisions |
| **Webhooks** | Subscribe to lifecycle events, delivery management |
| **Users / Registry** | Identity registration, nick management, agent resolution |
| **Enterprise** | Orgs, workspaces, members, service accounts, quotas |
| **MCP** | Model Context Protocol tool adapter for AI assistants |

Full API reference: [axme-docs](https://github.com/AxmeAI/axme-docs)

---

## Examples

### Basic intent (API key only)

```typescript
import { AxmeClient } from "@axme/axme";

const client = new AxmeClient({ apiKey: process.env.AXME_API_KEY });

const intent = await client.createIntent({
  intent_type: "report.generation.v1",
  payload: { report_id: "rpt_001", format: "pdf" },
  owner_agent: "agent://report-service",
});

console.log(intent.intent_id, intent.status); // ... PENDING
```

### Multi-actor approval flow (API key + actor token)

```python
from axme import AxmeClient, AxmeClientConfig

# Service submits intent that requires human approval
service = AxmeClient(AxmeClientConfig(api_key="axme_sa_..."))
intent = service.create_intent({
    "intent_type": "contract.sign.v1",
    "payload": {"contract_id": "ctr_001"},
    "owner_agent": "agent://legal-service",
})

# Human approves via their actor token
approver = AxmeClient(AxmeClientConfig(
    api_key="axme_sa_...",
    actor_token="user_jwt_token",  # identifies the approving human
))
pending = approver.list_inbox(owner_agent="agent://legal/approver")
for item in pending.get("items", []):
    approver.approve_inbox_thread(item["thread_id"], {"note": "Approved"}, owner_agent="agent://legal/approver")
```

More in [axme-examples](https://github.com/AxmeAI/axme-examples).

---

## Install

```bash
# Python
pip install axme

# TypeScript
npm install @axme/axme

# Go
go get github.com/AxmeAI/axme-sdk-go@latest

# CLI
go install github.com/AxmeAI/axme-cli/cmd/axme@latest
```

---

## Links

- **Cloud platform**: [cloud.axme.ai](https://cloud.axme.ai)
- **Alpha signup**: [cloud.axme.ai/alpha](https://cloud.axme.ai/alpha)
- **API docs**: [axme-docs](https://github.com/AxmeAI/axme-docs)
- **Protocol spec**: [axme-spec](https://github.com/AxmeAI/axme-spec)
- **Contact**: [hello@axme.ai](mailto:hello@axme.ai)
- **Security**: [SECURITY.md](SECURITY.md)

---

## Contributing

This repository is the entry point — not the implementation. To contribute:

- **Protocol / schemas** → [axme-spec](https://github.com/AxmeAI/axme-spec)
- **Documentation** → [axme-docs](https://github.com/AxmeAI/axme-docs)
- **SDK improvements** → respective SDK repository
- **Examples** → [axme-examples](https://github.com/AxmeAI/axme-examples)
- **Conformance checks** → [axme-conformance](https://github.com/AxmeAI/axme-conformance)

See [CONTRIBUTING.md](CONTRIBUTING.md) · [SECURITY.md](SECURITY.md) · [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
