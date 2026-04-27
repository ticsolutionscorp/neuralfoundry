# AI Knowledge NAS — Visualization Framework and Knowledge Explorer

## Product family

Defines [[neural-lens|Neural Lens]], the visualization module of [[neural-foundry|Neural Foundry]].

**Date:** 2026-04-26
**Status:** Ideation / architecture direction

## Core decision

The system should include an optional visualization module, but it should be designed as a **declarative visualization framework** rather than a hardcoded UI.

The default human-facing module should be a **Knowledge Explorer** built on top of the same APIs, commands, permissions, and review workflows that external agents use.

Headless does not mean invisible. Humans need a way to inspect what the system is building so they can trust it, catch false assumptions, and correct graph/ontology/entity-resolution errors.

## Product framing

The visualization layer should be:

- optional
- API-driven
- permission-aware
- agent-configurable
- declarative rather than arbitrary-code based
- usable for exploration, review, trust, and operational monitoring

Suggested module names:

- Knowledge Visualization Framework
- Knowledge Explorer
- Graph Explorer
- Semantic Explorer

Recommended framing:

> The visualization layer is a declarative, agent-configurable inspection framework. The default Knowledge Explorer is one client built on top of it.

## Architecture

```text
Headless Knowledge Core
  - entities
  - graph
  - ontology
  - rules
  - KPIs
  - evidence
  - review tasks
  - permissions

Visualization API
  - view specifications
  - saved views
  - permission checks
  - data queries
  - render hints
  - view events/actions

Default Knowledge Explorer
  - graph canvas
  - ontology map
  - evidence panel
  - entity resolution panel
  - rule/decision logic panel
  - review task actions

Custom Agent Views
  - maps
  - charts
  - timelines
  - KPI dashboards
  - graph neighborhoods
  - rule/explanation flows
  - dataset/model lineage views
```

The visual explorer should not own graph state. It should render views over the authoritative stores:

```text
Neo4j / Postgres / Qdrant / provenance store
        ↓
Semantic Graph API
        ↓
Visualization Framework / Knowledge Explorer
```

## Default Knowledge Explorer

The bundled default explorer should support focused inspection, not loading the entire graph at once.

Core capabilities:

- search by entity, concept, rule, document, source system, KPI, or dataset
- graph canvas with neighborhood expansion
- double-click node to expand/drill into details
- click node/edge to open details side panel
- filter by entity type, relationship type, confidence, review status, source, sensitivity, tenant, and rights
- evidence and citation panel
- source document/source span preview
- entity resolution panel
- ontology/rule explanation panel
- review task creation and status
- "show why this exists"
- "show source evidence"
- "flag as wrong"
- "create review task"

Important rule: **never render the whole graph by default**. Start from a search result, entity, rule, dataset, or scenario and expand outward one to two hops.

## Focused default views

### 1. Ontology map

Shows the business model:

- entity types
- relationship types
- concepts
- lifecycle states
- KPIs
- policies
- rules

Example:

```text
Customer → has_contract → Contract
Customer → owns_ticket → Ticket
LatePayment → is_a → RiskSignal
RiskSignal → contributes_to → CustomerHealth
```

### 2. Entity instance graph

Shows real-world entities and their relationships:

```text
Joe Smith
  → works_for → Acme Corp
  → owns → Ticket #123
  → signed → Contract #456
  → has_risk_signal → LatePayment
```

### 3. Evidence/provenance view

For any node, edge, rule, or concept, show:

- source documents
- source spans/chunks
- parser/model used
- confidence
- reviewer
- approval state
- last updated
- permission/training rights

### 4. Entity resolution view

For a canonical entity, show:

- aliases
- identifiers
- source records
- approved matches
- rejected matches
- candidate matches needing review
- merge/split history
- confidence and evidence

### 5. Rule / decision logic view

For business rules and decision logic, show:

- conditions
- actions
- affected entities/concepts
- source evidence
- confidence
- review/approval state
- effective dates
- downstream workflows/datasets/models that depend on the rule

### 6. Dataset governance / lineage view

Shows lineage from:

```text
sources → parsed artifacts → chunks/entities/rules → examples → datasets → models
```

Useful for audit, revocation, and model dependency review.

## Agent-configurable custom views

External agents should be able to define views using a structured specification.

Agents define intent, constraints, entities, metrics, and visual primitives. They should **not** submit arbitrary frontend code.

Example view spec:

```json
{
  "view_type": "graph_dashboard",
  "title": "At-Risk Enterprise Customers",
  "scope": {
    "tenant_id": "tenant_abc",
    "permission_mode": "viewer_effective_permissions"
  },
  "entities": {
    "types": ["Customer", "Contract", "Ticket", "Invoice", "RiskSignal"],
    "filters": {
      "customer.segment": "Enterprise",
      "customer.health": ["AtRisk", "Watch"]
    }
  },
  "relationships": [
    "has_contract",
    "owns_ticket",
    "has_invoice",
    "has_risk_signal"
  ],
  "kpis": [
    "open_high_priority_tickets",
    "overdue_invoice_total",
    "renewal_days_remaining",
    "customer_health_score"
  ],
  "visuals": [
    {
      "type": "graph",
      "layout": "force",
      "max_hops": 2
    },
    {
      "type": "bar_chart",
      "metric": "overdue_invoice_total",
      "group_by": "account_owner"
    },
    {
      "type": "timeline",
      "events": ["ticket_created", "invoice_due", "renewal_date"]
    }
  ],
  "actions": [
    "open_evidence",
    "explain_relationship",
    "create_review_task",
    "export_view"
  ]
}
```

## Supported visual primitives

### V1 primitives

- graph neighborhood
- ontology map
- node/edge detail panel
- KPI cards
- table
- bar chart
- line chart
- timeline
- evidence/citation panel
- review queue panel
- entity resolution panel
- rule explanation panel

### Later primitives

- geographic map
- Sankey / flow diagram
- rule / decision tree
- swimlane workflow
- heatmap
- hierarchy/tree
- comparison view
- simulation/scenario view
- dataset/model lineage explorer

## View actions

Views should expose safe, permission-checked actions such as:

- open entity
- expand graph
- open source evidence
- explain relationship
- explain rule
- trace provenance
- create review task
- flag assumption as wrong
- request more evidence
- approve/reject review task, if authorized
- export view, if permitted
- save view
- share view

Actions should call the same APIs used by agents.

## Agent-facing visualization API

Possible commands:

- `create_view(spec)`
- `validate_view_spec(spec)`
- `render_view(view_id)`
- `save_view(view_id, name, scope)`
- `list_views(filters)`
- `get_view(view_id)`
- `update_view(view_id, patch)`
- `delete_view(view_id)`
- `expand_view_node(view_id, node_id, options)`
- `get_view_evidence(view_id, node_or_edge_id)`
- `create_review_task_from_view(view_id, target_id, reason)`

## Technology options

Recommended V1:

- **Next.js / React** for the optional client.
- **Sigma.js** or **Cytoscape.js** for graph visualization.
- **React Flow** for ontology maps, rule flows, and cleaner directed diagrams.
- Backend visualization API over Neo4j/Postgres/Qdrant.
- Server-side query limits and graph expansion controls.

Shortcut / comparison tools:

- **Neo4j Bloom**: useful for internal exploration, but less productizable/custom.
- **Graphistry**: powerful large-graph visualization, external/platform-heavy.
- **Linkurious**: enterprise graph investigation, likely overkill/costly.
- **Kùzu Explorer**: interesting if Kùzu becomes part of the stack.

Recommendation: build a thin custom explorer and keep third-party visual tools optional.

## Permission and safety requirements

Visualization must obey the same permission model as retrieval and review.

Requirements:

- apply effective user/agent permissions before querying/rendering
- never show hidden nodes indirectly through graph layout or aggregate counts unless allowed
- filter evidence and source previews by permission
- enforce data-use rights on export/share actions
- audit view creation, access, export, and review actions
- prevent arbitrary code execution in agent-defined views
- validate all view specs against an allowlisted schema
- limit graph expansion to prevent accidental sensitive discovery or performance issues

## Design principles

1. Headless core remains authoritative.
2. Visualization is optional but important for trust.
3. Default Knowledge Explorer ships out of the box.
4. Agents can create scenario-specific views through declarative specs.
5. Views must be permission-aware and audit-backed.
6. Do not render the entire graph by default.
7. Use visualizations to support inspection, correction, review, and decision trust.
8. Avoid arbitrary generated frontend code; use controlled visual primitives.

## Relationship to the headless model

The visualization framework does not contradict the headless architecture. It is a client of the headless core.

```text
Headless Core
  - APIs
  - commands
  - events
  - review service

Optional Human Surfaces
  - Knowledge Explorer
  - custom agent-defined views
  - approval/chat integrations
  - admin console
```

This allows humans to inspect and trust the system while preserving agent-first operation.
