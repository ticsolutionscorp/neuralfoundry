# Neural Foundry — V1 Design Specification

**Status:** Draft for Nick Approval  
**Date:** 2026-04-27  
**Project:** Neural Foundry  
**Repo:** ticsolutionscorp/neuralfoundry  

---

## 1. Overview

Neural Foundry is an AI-native platform for organizational knowledge, memory, semantics, visualization, and governed model adaptation. It turns fragmented organizational data into a governed, explainable, agent-accessible business knowledge model.

The V1 target combines:

1. A personal OpenClaw memory appliance
2. A small-business knowledge base

Neural Vault makes OpenClaw agents materially smarter by providing source-backed recall, structured notes/tables, memory scopes, semantic context, entity resolution, and task-specific context packs.

### Product Family

| Module | V1 Role | Description |
|---|---|---|
| **Neural Vault** | Full implementation | Knowledge appliance, memory substrate, semantic graph, context packs, entity resolution, ontology, review service |
| **Neural Lens** | Full implementation | Optional visualization framework and Knowledge Explorer |
| **Neural Forge** | Architecture placeholder | Governed dataset and model adaptation layer (interfaces + docs only) |

### Core Architecture Split

- **AgentOS** manages *who* agents are: identity, registry, roles, capabilities, communication, status, discovery.
- **Neural Vault** manages *what* agents know: memory stores, source-backed knowledge, context retrieval, graph, ontology, rules, permissions, context packs.

> AgentOS tells us who the agent is and what it can do. Neural Vault tells the agent what it needs to know.

### Design Principles

1. **Headless first.** Agents operate all major functions through APIs, commands, events, and workflows. UI is optional.
2. **Human review is structured.** Humans participate through review tasks, approvals, evidence packages, and optional visual surfaces.
3. **Living semantics.** The system builds and maintains a versioned, evidence-backed ontology of how the business works.
4. **Permission-aware by default.** Every search, graph traversal, citation lookup, and generated answer is scoped to the requesting principal at retrieval time.
5. **Model-adapter architecture.** No hard dependency on one model family. Ship with sane defaults, make everything configurable.
6. **Derived artifacts inherit or restrict.** Never less restrictive than the source ACL.
7. **Deterministic policy enforcement.** LLMs never decide whether data can be used.

---

## 2. Deployment Model

### V1 Targets

- **Initial deployment:** Single team, expanding to multi-team within one company (not multi-org/multi-tenant)
- **Self-hosted:** Docker container with GPU passthrough for local inference
- **Cloud-deployable:** Azure and AWS with managed object storage, optional GPU workers
- **Single software stack** across all hardware tiers — no separate "Lite" architecture

### Hardware Tiers

| Tier | Name | GPU | RAM | Use Case |
|---|---|---|---|---|
| -1 | Personal Lite | 12–16 GB VRAM | 64 GB | One personal agent |
| 0 | CPU-only Dev | None | 32–64 GB | Prototype, text-only |
| 1 | Small Appliance | 16–24 GB VRAM | 96–128 GB | Few agents, small business |
| 2 | Recommended V1 | 24–32 GB VRAM | 128–256 GB | Serious self-hosted |
| 3 | Premium | RTX PRO 6000 96 GB / DGX Spark | 256–512 GB | Large models, heavy multimodal |
| 4 | Cluster | Dedicated GPU workers | 512 GB+ | Enterprise scale |

### Docker Compose Stack

Neural Vault runs as a set of containers:

```
neural-vault-api        — REST API + CLI server
neural-vault-workers    — BullMQ workers (ingestion, extraction, review)
neural-vault-postgres   — PostgreSQL (metadata, permissions, ontology, review, audit)
neural-vault-qdrant     — Qdrant (vector index)
neural-vault-neo4j      — Neo4j (graph DB)
neural-vault-seaweedfs  — SeaweedFS (object/file storage, default backend)
neural-vault-redis      — Redis (BullMQ job queue + cache)
neural-vault-ollama     — Ollama or vLLM (local model serving, optional)
neural-lens             — Next.js Knowledge Explorer (optional)
```

An optional n8n container can be added for deployment-specific repeatable workflow tasks, but it is not an internal dependency.

---

## 3. Data Architecture

### Storage Components

| Store | Role | Contents |
|---|---|---|
| **PostgreSQL** | Metadata, permissions, ontology, rules, entity registry, jobs, review, audit, dataset governance | Principals, groups, ACLs, ontology definitions, entity types, rule versions, review tasks, job state, rights registry, dataset metadata, audit logs |
| **SeaweedFS** (or S3-compatible backend) | Object/file storage | Original files, parsed artifacts, chunks, transcripts, OCR outputs, thumbnails, derived media |
| **Qdrant** | Vector index | Embeddings with payload metadata (tenant_id, source_id, acl_groups, sensitivity, model version) |
| **Neo4j** | Graph DB | Canonical entities, relationships, extracted mentions, provenance edges, ontology-to-instance links, inferred relationships |
| **Redis** | Job queue + cache | BullMQ job state, session cache |

### Storage Abstraction

Build against an S3-compatible storage abstraction. Never bind to one backend.

| Profile | Backend | Use Case |
|---|---|---|
| Default self-hosted | SeaweedFS | V1 production default |
| Experimental | RustFS | Parallel PoC evaluation |
| Edge/geo | Garage | Lightweight distributed |
| Cloud | AWS S3 / Azure Blob | Cloud deployments |
| Dev | Local filesystem | Development only |

### Data Flow Layers

```
Storage Layer         → files, objects, source references, checksums, versions
Parsing Layer         → text, tables, OCR, transcripts, captions, document structure
Knowledge Layer       → chunks, embeddings, extracted entities, relationships, graph edges
Entity Resolution     → canonical entities, aliases, source IDs, merge/split decisions
Semantic Layer        → entity types, relationship types, concepts, states, rules, decision logic
Retrieval + Agent     → query understanding, GraphRAG, context packs, review tasks
```

---

## 4. Ingestion & Parsing

### V1 Connectors

- Local filesystem folders (watched, with create/update/delete events via fswatch/inotify/FSEvents)
- Obsidian-like note folders (when present)
- OpenClaw workspace/project files
- Neural Vault native notes/tables

### Future Connectors (Post-V1)

- Google Drive / Workspace
- Microsoft Graph / SharePoint / OneDrive
- S3-compatible storage
- Slack / Teams / email
- SQL databases (read-only connectors)

### Parsing Pipeline

| Stage | Tool | Role |
|---|---|---|
| Primary parser | Docling | Structured document intelligence, layout, tables, OCR, GPU-accelerated |
| Fallback parser | Apache Tika | Broad file type detection/extraction for long-tail formats |
| OCR fallback | Tesseract / PaddleOCR | When Docling OCR is insufficient |
| Audio transcription | faster-whisper (large-v3-turbo) | Speech-to-text |
| Video processing | FFmpeg | Keyframe extraction + transcription |

### Supported File Types (V1)

- Documents: PDF, DOCX, PPTX, XLSX, HTML, Markdown, plain text, code files
- Images: JPEG, PNG, GIF, WebP, TIFF (with OCR/captioning)
- Audio: MP3, WAV, M4A, FLAC (with transcription)
- Video: MP4, MKV, AVI, MOV (keyframe extraction + transcription)

---

## 5. Model Strategy

### Architecture

Model-adapter pattern: every model role has an adapter interface. Ship with configurable profiles. Store model name/version/dimensions with every embedding; reindexing is a first-class maintenance operation.

### V1 Default Profile — Nemotron Stack

| Role | Default | Efficient Fallback |
|---|---|---|
| OCR | Nemotron OCR (via Docling) | Tesseract / PaddleOCR |
| Embeddings | Nemotron embedding family | Qwen3-Embedding-4B / BGE-M3 |
| Reranking | Nemotron reranker family | Qwen3-Reranker-4B / BGE-reranker-v2-m3 |
| VLM (hard pages, charts) | Nemotron VL | Qwen3-VL 8B / 32B |
| Metadata/entity extraction | Nemotron 3 Nano / Super | Qwen3 local LLM / GLM 5.1 |
| Answer/RAG | Nemotron 3 Nano / Super | Qwen3 MoE |
| Agentic/tool-heavy | Nemotron 3 Super | GLM 5.1 |

### V1 Alternative Profile — Qwen3 Stack (Non-NVIDIA Hardware)

| Role | Model |
|---|---|
| Embeddings | Qwen3-Embedding-4B (BGE-M3 efficient fallback) |
| Reranking | Qwen3-Reranker-4B (BGE-reranker-v2-m3 fallback) |
| VLM | Qwen3-VL 8B / 32B |
| Extraction | Qwen3 local LLM or GLM 5.1 |
| Answer | Qwen3 MoE |
| Agentic | GLM 5.1 |

Qwen3/BGE adapters serve as benchmark comparators and non-NVIDIA hardware fallback profiles.

### Serving Stack

- Simple: Ollama
- Production: vLLM / SGLang / TensorRT-LLM
- Multi-worker: NVIDIA Dynamo

---

## 6. Permissions & Security

### Core Principle

Permission-aware at retrieval time, not just at UI time. The system never retrieves unauthorized chunks into model context. Filtering after generation is not sufficient.

### Identity Model

Every request maps to a canonical principal:

```json
{
  "principal_id": "uuid",
  "principal_type": "human | agent | service_account",
  "tenant_id": "org_uuid",
  "external_identities": [
    { "provider": "entra_id", "id": "...", "email": "..." },
    { "provider": "google_workspace", "id": "...", "email": "..." },
    { "provider": "slack", "id": "..." }
  ],
  "groups": ["group_uuid"],
  "roles": ["role_name"],
  "attributes": { "department": "finance", "clearance": "confidential" }
}
```

### V1 Identity Providers

- Supabase Auth (standalone/small deployments)
- Enterprise OIDC (Entra ID, Google Workspace, Okta, Auth0)

### Authorization Model

Layered: Tenant → Source → Document → Chunk → Field-level/Sensitivity

Not RBAC-only. Use ACL + group membership + ABAC/policy rules.

**V1 Implementation:**
- PostgreSQL RLS for metadata tables
- App-layer policy checks in the API server
- Qdrant payload filters for vector retrieval (tenant_id, acl_group_ids, sensitivity, source_id)
- Neo4j graph queries scoped by tenant/source/ACL metadata

**V1 Avoid:**
- Per-user vector indexes
- Post-generation-only permission filtering
- Treating agents as global admins
- Caching answers without principal/scope keys

### Agent Permissions

Agents are first-class principals with scoped permissions. On-behalf-of mode evaluates the intersection of:
- What the human can see
- What the agent is allowed to do
- What the integration/source allows

This prevents agents from becoming permission escalation paths.

### Permission Propagation

Derived artifacts (chunks, embeddings, summaries, graph nodes/edges) inherit the source document ACL. Derived can be equal or more restrictive, never less.

### Answer Safety

- Include citations to authorized source chunks only
- If no authorized evidence exists, say so
- Do not mention hidden document titles or counts (leaks existence)
- Cache answers only with the principal/scope/policy hash

### Audit Logging

Log every security-relevant action: who/what asked, on behalf of whom, query text, filters/scopes, document IDs returned, citations used, answer generated, denied attempts, policy version. Immutable/append-only.

---

## 7. Memory & Knowledge Model

### Memory Scopes

| Scope | Description |
|---|---|
| global/org | Organization-wide canonical knowledge |
| project | Project-scoped knowledge |
| shared team | Team-shared knowledge |
| agent | Agent-specific working memory |
| user | User-specific preferences and history |
| session/episodic | Temporary session state |

### Memory States

```
ephemeral → proposed → reviewed → canonical → deprecated → revoked
```

- Agents write freely to working/proposed memory
- Shared/canonical knowledge requires evidence, confidence, policy, or review
- Memory promotion: `propose_fact` → `promote_memory` (with review if needed)

### Native Notes & Tables

Agents use Neural Vault directly for persistent structured state — no dependency on Obsidian/Notion for durable artifacts.

Supported types: project notes, decision logs, task state tables, recurring checklists, entity-specific notes, customer/vendor/account notes, structured operating tables, workflow state, review queues, context annotations.

### Entity Resolution

Bridge from extracted mentions to canonical real-world instances:

```
"Joe Smith" in CRM →
"Joseph Smith" in email →
"J. Smith" in invoice PDF →
joe.smith@acme.com in support ticket →
  canonical entity: Person:PERS-1042
```

The graph distinguishes:
- Extracted mention nodes
- Source-system record nodes
- Canonical entity nodes
- Alias/identifier nodes
- Evidence links with confidence and review status

### V1 Entity Types (Generic)

- Person
- Organization
- Document
- Project
- Location
- Event
- Topic/Concept

Domain-specific types (Customer, Vendor, Contract, Invoice, etc.) added in future versions.

### Living Ontology

Defines business vocabulary and constraints:

- Entity types with fields, identifiers, lifecycle states, relationships
- Relationship types with allowed properties
- Business concepts and concept hierarchies
- Lifecycle states and business events
- Metrics and KPIs
- Source/evidence links
- Confidence and review state
- Versioning with effective dates

Living workflow:
1. Ingestion/parsing extract candidate entities, relationships, concepts, rules
2. Entity resolution proposes canonical entities and merge/split decisions
3. System proposes ontology updates and business rule candidates with evidence
4. Review tasks created for uncertain or high-impact proposals
5. Agents summarize evidence; authorized humans decide via Slack/Telegram/etc.
6. Review service validates authority, records decision, emits events
7. Approved ontology improves future extraction, retrieval, and context packs

### Business Rules / Decision Logic

Captures the "why": policies, SOPs, heuristics, exceptions, decision criteria.

Each rule has:
- Conditions (when)
- Actions (then)
- Source evidence
- Confidence score
- Review status
- Effective dates
- Scope (tenant/client/department)
- Sensitivity and training/export rights

Rules support both deterministic rules and evidence-backed inferred heuristics. Inferred heuristics support explanations and recommendations until approved.

### Context Pack Builder (V1 — Simpler Version)

Assembles task-specific context from:
- Relevant chunks (vector retrieval)
- Entity summaries
- Applicable rules

V2 will add: decisions, full source citations with confidence markers, recent events.

---

## 8. Review & Approval Service

### Model

Review tasks as first-class objects with structured evidence, confidence, risk, authority validation, and full audit trail.

### Review Task Schema

```
ReviewTask:
  id, type, subject_id, tenant_id
  proposed_action (merge, split, approve, reject, freeze, etc.)
  evidence_refs (doc chunks, source records, graph edges)
  confidence, risk_level
  required_reviewer_role
  status: pending | approved | rejected | needs_more_evidence | escalated | expired
  assigned_to, decision, decided_by, decided_at
  audit_log[]
```

### Review Task Types

| Category | Types |
|---|---|
| Entity Resolution | merge, split, reject match, approve alias, declare source-of-truth |
| Ontology/Semantic | approve entity type, relationship type, concept, rule, inferred relationship; retire/replace |
| Dataset Governance | approve training eligibility, dataset freeze, export |
| Security/Compliance | approve boundary crossing, revocation, policy exception |

### Agent-Facing Review API

`list_review_tasks`, `get_review_task`, `explain_review_task`, `request_more_evidence`, `approve_review_task`, `reject_review_task`, `bulk_approve`, `assign_review_task`, `escalate_review_task`, `subscribe_review_events`, `get_review_audit_log`

### Event-Driven Workflow

Events: `review_task.created`, `assigned`, `updated`, `approved`, `rejected`, `escalated`, `expired`, `needs_more_evidence`

These trigger agent notifications, Slack/Telegram messages, downstream graph updates, dataset rebuilds.

### Human Interaction Pattern

1. System creates review task
2. Agent receives event and summarizes evidence for human
3. Human replies in Slack/Telegram: "Approve the first two, reject the third"
4. Agent converts reply into structured review decisions
5. Review service validates human authority
6. Decision recorded with audit trail
7. Downstream updates triggered

### Authority Boundary

Agents prepare and explain; humans decide; policy engine validates authority. The system verifies reviewer identity, checks authority/role/scope, and rejects unauthorized approvals.

---

## 9. API Surface

### V1: REST + CLI + commands.json

No GraphQL in V1. Add in V2 when query patterns are proven.

### Agent Command Set

**Notes & Tables:**
`create_note`, `update_note`, `search_notes`, `link_note_to_entity`, `create_table`, `upsert_table_row`, `query_table`, `link_table_row_to_entity`

**Memory:**
`store_memory`, `propose_fact`, `promote_memory`

**Context:**
`build_context_pack`

**Entity & Semantic:**
`resolve_entity`, `get_entity`, `get_entity_type`, `find_entities`, `get_relationships`, `explain_concept`, `explain_rule`, `trace_evidence`, `find_rules_applicable_to`

**Search & Retrieval:**
`search(query, filters)`, `get_document(id)`, `get_chunk(id)`, `explain_citation(id)`, `traverse_graph(entity)`

**Review:**
`list_review_tasks`, `get_review_task`, `explain_review_task`, `request_more_evidence`, `approve_review_task`, `reject_review_task`, `bulk_approve`, `assign_review_task`, `escalate_review_task`, `subscribe_review_events`, `get_review_audit_log`

**Visualization:**
`create_view(spec)`, `validate_view_spec(spec)`, `render_view(view_id)`, `save_view`, `list_views`, `get_view`, `expand_view_node`

**Dataset Governance (V1 foundation):**
`register_rights`, `get_rights`, `trace_provenance`, `create_dataset_draft`, `freeze_dataset`

### Interface Protocols

- REST API with OpenAPI spec
- CLI with structured output (JSON)
- `commands.json` machine-readable manifest
- Webhook/event bus for review events
- CLI + REST are the primary surfaces; webhooks for async integration

---

## 10. Orchestration

### V1

- **BullMQ** (backed by Redis) for internal job queues: ingestion, parsing, embedding, extraction, review routing, retries
- **n8n** available as a separate deployment-specific instance for defined, repeatable workflow tasks — not an internal dependency of Neural Vault

### V2 Consideration

- **Temporal** for complex long-running workflows requiring durable state, saga patterns, and long waits

### Ingestion Job Lifecycle

```
file detected → enqueue parse job → parse → enqueue embed job → embed
  → enqueue extract job → extract entities/relationships → enqueue graph job
  → write to graph → check entity resolution → create review tasks if uncertain
  → emit events
```

Each stage is a BullMQ job with retries, dead-letter handling, and audit logging.

---

## 11. Dataset Governance (V1 Foundation)

V1 includes the governance layer that Neural Forge will build on, but not the full pipeline.

### V1 Scope

- **Rights registry:** Per-source/document/chunk flags: `can_index`, `can_retrieve`, `can_summarize`, `can_extract_entities`, `can_train`, `can_export`, `can_leave_boundary`, retention, license, data owner, client boundary
- **Provenance graph:** Source → parsed artifacts → chunks → entities → examples → datasets, with model versions and parser info
- **Basic dataset builder:** Filter, deduplicate, split train/val/test, freeze versions, generate dataset cards

### Post-V1

- Redaction/anonymization pipeline
- Review queues for training eligibility and example approval
- Quality scoring (duplication, source diversity, OCR confidence, contradiction clusters)
- Evaluation set management
- Full Neural Forge training pipeline

---

## 12. Neural Lens — Visualization

### V1: Full Knowledge Explorer

Built as an optional Next.js/React client consuming the same headless APIs.

### Default Views

| View | Description |
|---|---|
| Graph neighborhood | Expand from entity/search, 1–2 hops, never render whole graph |
| Ontology map | Entity types, relationship types, concepts, lifecycle states |
| Entity instance graph | Real-world entities and their relationships |
| Evidence/provenance | Source documents, spans, parser/model, confidence, approval state |
| Entity resolution | Aliases, source records, approved/rejected/candidate matches |
| Rule/decision logic | Conditions, actions, affected entities, evidence, review state |
| KPI cards + tables/charts | Metrics dashboards |
| Timeline | Events by entity/project |
| Review queue panel | Pending tasks with evidence and actions |

### Declarative View Framework

Agents define views via structured specs (not arbitrary code):

```json
{
  "view_type": "graph_dashboard",
  "title": "At-Risk Enterprise Customers",
  "scope": { "tenant_id": "...", "permission_mode": "viewer_effective_permissions" },
  "entities": { "types": ["Person", "Organization"], "filters": {} },
  "relationships": ["works_for", "owns_ticket"],
  "kpis": ["open_tickets", "risk_score"],
  "visuals": [
    { "type": "graph", "layout": "force", "max_hops": 2 },
    { "type": "bar_chart", "metric": "open_tickets", "group_by": "owner" }
  ],
  "actions": ["open_evidence", "create_review_task"]
}
```

### Technology

- Next.js / React
- Sigma.js or Cytoscape.js (graph visualization)
- React Flow (ontology maps, rule flows, directed diagrams)

### Safety Requirements

- Apply effective permissions before querying/rendering
- Never show hidden nodes indirectly through layout or aggregate counts
- Filter evidence and source previews by permission
- Audit view creation, access, export, and review actions
- Validate all view specs against an allowlisted schema
- Limit graph expansion to prevent accidental sensitive discovery

---

## 13. Neural Forge — V1 Placeholder

Architecture stub in code with interfaces only. No functional implementation.

Defines interfaces for:
- Dataset governance (consumed by V1 foundation)
- Model adaptation pipeline (future: fine-tuning, distillation, model registry)
- Training job management

Full implementation deferred to V2+.

---

## 14. V1 Scope Summary

### In Scope

- Neural Vault: full knowledge appliance with ingestion, parsing, embedding, vector search, graph, entity resolution, living ontology, business rules, native notes/tables, memory scopes, context pack builder (simpler V1), review/approval service
- Neural Lens: full Knowledge Explorer with declarative view framework
- Neural Forge: architecture placeholder (interfaces + docs only)
- Dataset governance foundation: rights registry, provenance, basic dataset builder
- Permissions: Postgres RLS + app-layer policy + Qdrant filters + Neo4j ACL metadata
- Connectors: local folders, Obsidian folders, OpenClaw workspace, Neural Vault native
- API: REST + CLI + commands.json + webhooks
- Orchestration: BullMQ (internal), n8n (optional external)
- Models: Nemotron default profile, Qwen3 alternative profile (non-NVIDIA), model-adapter architecture
- Deployment: Docker Compose, self-hosted (GPU passthrough) or cloud (Azure/AWS)
- Storage: SeaweedFS (default) + S3-compatible abstraction

### Explicitly Post-V1

- GraphQL API
- Cloud SaaS connectors (Google Workspace, SharePoint, S3, Slack, SQL)
- Domain-specific entity types
- Full dataset governance pipeline (redaction, review queues, quality scoring, eval sets)
- Full context pack builder (decisions, confidence markers, recent events)
- Temporal workflow orchestration
- Neural Forge functional implementation
- MCP server
- Video scene understanding
- Retention policies
- Write-back to source systems
- RDF/OWL export, SHACL validation
- Multi-tenant / multi-org

---

## 15. Open Items (Lower Priority)

These can be refined during build:

- Neural Forge placeholder depth (stub in code vs. docs only)
- MCP server timing (confirm post-V1)
- Video scene understanding timing (confirm post-V1)
- Redaction pipeline timing (defer with full dataset governance)
- RustFS PoC timing
- LlamaIndex / LangChain integration patterns for extraction workflows
