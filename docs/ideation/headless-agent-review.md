# AI Knowledge NAS — Headless Agent-First Review and Human-in-the-Loop Workflows

## Product family

Core headless review layer for [[neural-vault|Neural Vault]], [[neural-forge|Neural Forge]], and [[neural-lens|Neural Lens]]. Parent: [[neural-foundry|Neural Foundry]].

**Date:** 2026-04-26
**Status:** Ideation / architecture direction

## Core decision

The system should be **headless by default**. External agents should be able to operate every major function, including review workflows, approvals, entity resolution decisions, ontology updates, dataset governance, exports, and model-training approvals.

Human-in-the-loop should be implemented as a governed review/approval service, not as a mandatory human-facing application UI.

A web console can exist later for bulk/admin work, but it should not be the primary integration surface.

## Why this matters

The product is intended to be operated by agents as much as by humans. Agents should not only query knowledge; they should be able to coordinate ingestion, inspect entity-resolution candidates, request human decisions, manage review queues, submit ontology proposals, build datasets, and trigger approved downstream workflows.

Humans can participate through:

- Slack / Teams / Telegram / email
- agent-mediated conversations
- approval links or signed actions
- optional thin admin/review console
- customer-specific workflow tools

The core system should expose structured APIs and events so any agent or integration surface can participate.

## Review service model

Human review is represented as first-class review tasks.

Example review task schema:

```yaml
ReviewTask:
  id: review_123
  type: entity_merge | entity_split | ontology_change | rule_approval | dataset_freeze | export_approval | training_approval | revocation_review
  subject_id: entity_candidate_456
  tenant_id: tenant_abc
  proposed_action:
    action: merge_entities
    target: Customer:CUST-1042
    candidates:
      - mention_1
      - source_record_2
  evidence_refs:
    - doc_chunk_1
    - crm_record_88
    - graph_edge_22
  confidence: 0.87
  risk_level: medium
  required_reviewer_role: data_steward
  status: pending | approved | rejected | needs_more_evidence | escalated | expired
  assigned_to: user_or_role
  decision:
    outcome: approved
    reason: "CRM ID and email domain match."
  decided_by: user_789
  decided_at: 2026-04-26T21:00:00Z
  audit_log: [...]
```

## Agent-facing review API

The review system should expose commands such as:

- `list_review_tasks(filters)`
- `get_review_task(id)`
- `explain_review_task(id)`
- `request_more_evidence(id)`
- `approve_review_task(id, decision)`
- `reject_review_task(id, reason)`
- `bulk_approve(task_ids, decision)`
- `assign_review_task(id, user_or_role)`
- `escalate_review_task(id, reason)`
- `subscribe_review_events(filters)`
- `get_review_audit_log(id)`

These should be available through:

- REST API / OpenAPI
- `commands.json`
- CLI
- webhook/event bus
- optional MCP server or agent skill

## Human interaction pattern

Agents prepare and explain decisions; humans approve or reject; the policy engine validates authority; the review service records the decision.

Example flow:

```text
1. Entity resolution pipeline finds 3 possible duplicate customer records.
2. System creates ReviewTask(type=entity_merge, risk=medium).
3. Agent receives event and summarizes the evidence for a human reviewer.
4. Human replies in Slack/Teams/Telegram: "Approve the first two, reject the third."
5. Agent converts the reply into structured review decisions.
6. Review service validates that the human has authority.
7. Decision is recorded with full audit trail.
8. Entity graph updates; downstream ontology/dataset workflows are notified.
```

## Authority boundary

Agents are not the authority boundary.

Agents may:

- prepare evidence
- recommend actions
- explain confidence and risk
- ask humans for decisions
- submit decisions on behalf of authenticated users when the channel supports it
- execute approved actions

The system must:

- verify reviewer identity
- check reviewer authority/role/scope
- enforce policy rules
- record the exact decision and evidence package
- reject unauthorized approvals
- preserve audit history

## Review task types

Initial review task categories:

### Entity resolution

- merge entities
- split entity
- reject candidate match
- approve alias/identifier
- declare source-of-truth record
- mark entity as duplicate / obsolete / renamed

### Ontology and semantic layer

- approve entity type
- approve relationship type
- approve concept definition
- approve business rule
- approve inferred relationship
- scope rule to tenant/client/department
- retire or replace ontology item

### Dataset governance

- approve source collection for training eligibility
- approve generated Q&A/instruction examples
- approve redaction policy result
- freeze dataset version
- approve dataset export
- approve fine-tune/training job
- approve model deployment

### Security/compliance

- approve data boundary crossing
- approve revocation action
- approve quarantine/retirement of affected datasets/models
- review sensitive-content classification
- review policy exception

## Event-driven workflow

The system should publish events for agents and workflow engines:

- `review_task.created`
- `review_task.assigned`
- `review_task.updated`
- `review_task.approved`
- `review_task.rejected`
- `review_task.escalated`
- `review_task.expired`
- `review_task.needs_more_evidence`

These events can trigger:

- agent notifications
- Slack/Teams/Telegram messages
- n8n/Temporal workflows
- downstream graph updates
- dataset rebuilds
- model registry status changes

## Recommended technology approach

### V1

- **PostgreSQL**: review tasks, assignments, decisions, comments, authority, audit logs.
- **App-layer policy checks**: validate reviewer authority and scope.
- **REST/OpenAPI + commands.json**: agent-facing control surface.
- **Webhook/event system**: publish review events to agents/workflows.
- **n8n or BullMQ**: lightweight review workflow orchestration and retries.
- **Slack/Teams/Telegram/email adapters**: human notification and response surfaces.
- **Optional thin console**: bulk review/admin only, not required for core usage.

### Later

- **Temporal** for complex long-running review workflows and durable waiting states.
- **OPA/Rego or Cedar** for richer policy/authority evaluation.
- **MCP server / ClawHub skill** for standardized agent operations.
- **Signed approval links** for non-chat channels.
- **Fine-grained delegation and approval chains** for enterprise customers.

## Design principles

1. Headless by default.
2. Agent-first operation for all functions, not only search/query.
3. Human-in-the-loop as structured review tasks and approval decisions.
4. Agents prepare and explain; humans decide; policy engine validates authority.
5. Optional UI is secondary and should consume the same APIs as agents.
6. Every review decision must be auditable and tied to evidence.
7. Review tasks should be event-driven so external agents and workflows can coordinate around them.

## Product architecture impact

The interface layer should be described as:

```text
Agent Control Plane
  - REST API / OpenAPI
  - commands.json
  - CLI
  - webhooks/events
  - optional MCP / skill

Review and Approval Service
  - review task queue
  - evidence packages
  - human decisions
  - policy checks
  - audit logs
  - event publication

Optional Human Surfaces
  - Slack / Teams / Telegram / email approval flows
  - thin admin/review console
  - graph/ontology visualization tools
```

This makes the system suitable for OSForge-style operation where agents coordinate work, humans approve key decisions, and the product remains deployable in environments that do not want a heavy UI dependency.
