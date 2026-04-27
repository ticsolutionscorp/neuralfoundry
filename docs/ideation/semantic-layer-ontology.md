# AI Knowledge NAS — Semantic Layer, Living Ontology, and Business Logic

## Product family

Core to [[neural-vault|Neural Vault]], visualized by [[neural-lens|Neural Lens]], and used by [[neural-forge|Neural Forge]]. Parent: [[neural-foundry|Neural Foundry]].

**Date:** 2026-04-26
**Status:** Ideation / architecture direction

## Core idea

The AI Knowledge NAS should not only store and retrieve organizational knowledge. It should gradually build a machine-readable model of how the business works.

A living ontology is required to capture business context beyond raw documents and search results:

- what key entities exist
- what types those entities belong to
- how entities relate to each other
- what business concepts mean
- what rules, policies, and decision logic govern the organization
- why certain decisions are made
- what evidence supports those meanings and rules

The ontology should be **living, evidence-backed, versioned, and reviewable**, not a static schema maintained only by admins.

## Conceptual layering

```text
Storage layer
  → files, objects, source references, checksums, versions

Parsing / extraction layer
  → text, tables, OCR, transcripts, captions, document structure

Knowledge layer
  → chunks, embeddings, extracted entities, extracted relationships, graph edges

Entity resolution layer
  → canonical real-world entities, aliases, source IDs, merge/split decisions

Semantic layer / living ontology
  → entity types, relationship types, business concepts, states, metrics, policies, rules, decision logic

Retrieval + agent + dataset layers
  → query understanding, GraphRAG, business explanations, training examples, eval cases, automation inputs
```

Simple framing:

- Storage tells us what exists.
- Parsing tells us what the content says.
- Entity resolution tells us what real-world things the content refers to.
- Ontology tells us what those things mean.
- Decision logic explains why the business behaves the way it does.

## Required semantic modules

### 1. Entity type model

Defines business object types and their expected fields, identifiers, lifecycle states, and relationships.

Examples:

- Customer
- Vendor
- Employee
- Product
- Contract
- Invoice
- Project
- Asset
- Ticket
- Location
- Policy
- Risk
- Recommendation
- Workflow
- Decision
- KPI

Example entity type definition:

```yaml
EntityType: Customer
Description: External party that buys or subscribes to products/services.
Fields:
  - customer_id
  - legal_name
  - common_name
  - email_domains
  - status
  - segment
  - account_owner
Identifiers:
  - CRM customer ID
  - billing system account ID
  - email domain
Relationships:
  - has_contract
  - placed_order
  - owns_ticket
  - belongs_to_segment
  - has_risk_score
LifecycleStates:
  - prospect
  - active
  - at_risk
  - churned
Sensitivity: business_confidential
```

### 2. Entity resolution-linked graph

Entity resolution is the bridge from extracted mentions to canonical real-world instances.

Example:

```text
"Joe Smith" in CRM
"Joseph Smith" in email
"J. Smith" in invoice PDF
joe.smith@acme.com in support ticket
→ canonical entity: Customer:CUST-1042
```

The graph should distinguish between:

- extracted mention nodes
- source-system records
- canonical entity nodes
- alias/identifier nodes
- evidence links
- confidence and review status

This avoids a noisy graph where every spelling variant becomes a separate customer.

### 3. Ontology manager

Defines the business vocabulary and constraints.

Includes:

- entity types
- relationship types
- allowed properties
- lifecycle states
- business events
- metrics and KPIs
- concept hierarchies
- inferred classes
- source/evidence links
- confidence and review state

Example ontology statements:

```text
Customer is a type of ExternalParty.
Invoice is a type of FinancialDocument.
LatePayment is a type of RiskSignal.
ChurnRisk may be inferred from support volume, payment delay, usage decline, sentiment, and renewal proximity.
Escalation is a workflow event triggered by a business rule.
```

### 4. Business rules / decision logic layer

Captures the "why" of the business: policies, SOPs, heuristics, exceptions, and decision criteria.

Examples:

- approval rules
- pricing logic
- escalation paths
- prioritization logic
- customer health scoring
- risk scoring
- exception handling
- compliance policies
- operational heuristics
- tacit knowledge captured from interviews/conversations

Example rule:

```yaml
Rule: Escalate delayed enterprise orders
When:
  - customer.segment = Enterprise
  - order.status = Delayed
  - delay_days > 3
Then:
  - create escalation
  - notify account_owner
  - update customer_health = AtRisk
Sources:
  - Operations SOP v4
  - VP Ops interview 2026-04-20
Confidence: 0.86
ReviewStatus: approved
EffectiveFrom: 2026-04-01
Scope: internal_only
```

The rules layer should support both deterministic rules and evidence-backed inferred heuristics. Deterministic rules can be used for automation; inferred heuristics should support explanations and recommendations until approved.

### 5. Semantic query API

The semantic layer should expose query primitives for humans, agents, and dataset generation.

Examples:

- `get_entity_type(name)`
- `get_entity(entity_id)`
- `resolve_entity(mention, context)`
- `find_entities(type, filters)`
- `get_relationships(entity_id, relationship_type)`
- `explain_concept(concept_id)`
- `explain_rule(rule_id)`
- `why_decision(decision_or_recommendation_id)`
- `trace_evidence(statement_id)`
- `find_rules_applicable_to(entity_or_scenario)`

## Living ontology workflow

The ontology should improve over time through a human-in-the-loop loop. Because the product is headless and agent-first, review should be implemented through structured review tasks and APIs, not a mandatory UI.

1. Ingestion and parsing extract candidate entities, relationships, concepts, and rules.
2. Entity resolution proposes canonical entities and merge/split decisions.
3. The system proposes ontology updates and business rule candidates with evidence.
4. Review tasks are created for uncertain or high-impact proposals.
5. Agents summarize evidence and ask authorized humans for decisions through Slack/Teams/Telegram/email or other configured channels.
6. Human reviewers approve, reject, correct, or scope the proposed semantics.
7. The review service validates reviewer authority, records the decision, and emits events.
8. Approved ontology improves future extraction, retrieval, query interpretation, and training dataset generation.
9. Ontology versions and rule versions preserve history and support auditability.

## Versioning and review

The ontology must be versioned because business meaning changes.

Track:

- ontology version
- entity type version
- relationship type version
- rule version
- effective dates
- retired/replaced states
- source evidence
- reviewer/approver
- confidence score
- tenant/client scope
- sensitivity and training/export rights

Example:

```text
CustomerHealthScoring v1.2
Active from: 2026-04-01
Derived from: CRM policy, support SOP, finance rules
Approved by: VP Customer Success
```

## Technical implementation direction

Use a pragmatic typed graph ontology first. Avoid starting with full OWL/RDF complexity unless a customer requires it.

Recommended storage split:

- **PostgreSQL**: ontology definitions, versions, approvals, rights, rule metadata, effective dates, review tasks, decisions, authority checks, and audit logs.
- **Neo4j**: canonical entity graph, relationships, provenance edges, inferred relationships, ontology-to-instance links.
- **Qdrant**: semantic retrieval over source evidence, rule explanations, ontology descriptions, business concepts, and examples.
- **Object storage**: original sources and derived artifacts.
- **Workflow/event layer**: review task events, approval routing, retries, escalation, and downstream graph/dataset updates.

Optional later:

- RDF/OWL export for enterprise semantic-web compatibility.
- SHACL-like validation for ontology constraints.
- Rule engine integration for deterministic automation where policies are formally approved.

## Relationship to retrieval

The semantic layer improves retrieval by expanding questions through business meaning rather than keywords only.

Example query: "Which customers are at risk?"

Without ontology:

- search for literal terms such as "risk", "churn", "complaint"

With ontology:

- understand Customer
- understand RiskSignal
- include overdue invoices, high-priority tickets, declining usage, negative sentiment, renewal proximity, executive complaints, and rule-defined customer health scoring
- return cited evidence and explain which rule/concept made each result relevant

## Relationship to dataset governance and model training

Ontology artifacts become high-value training/evaluation inputs:

- entity extraction examples
- relationship extraction examples
- entity resolution examples
- business reasoning examples
- rule-following examples
- classification examples
- workflow decision examples
- "explain why" examples

But ontology objects also need governance metadata:

- provenance
- source evidence
- sensitivity
- review status
- tenant/client scope
- `can_train`
- `can_export`
- `can_leave_boundary`

A business rule extracted from confidential board notes may be usable internally but not for a shared industry model.

## Design principles

1. Make the ontology living, evidence-backed, and reviewable.
2. Separate extracted mentions from canonical entities.
3. Link ontology definitions to graph instances and source evidence.
4. Version concepts, rules, and relationships over time.
5. Treat decision logic as first-class knowledge, not just free-text documentation.
6. Use deterministic governance and policy checks around ontology-derived training data.
7. Start pragmatic with typed graph semantics; add RDF/OWL compatibility only when justified.

## Product architecture placement

```text
Knowledge Core
  - ingestion
  - parsing
  - storage
  - vector index
  - graph index
  - permissions

Semantic Layer
  - entity registry
  - entity resolution
  - living ontology
  - business concepts
  - rules / decision logic
  - evidence and provenance
  - semantic query API

Dataset Governance
  - rights registry
  - redaction
  - dataset builder
  - approvals
  - eval sets

Model Foundry / Neural Forge
  - fine-tuning
  - distillation
  - model registry
```
