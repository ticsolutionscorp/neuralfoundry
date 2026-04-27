# AI Knowledge NAS — Initial Requirements

## Product family

Requirements for [[neural-foundry|Neural Foundry]] and its modules: [[neural-vault|Neural Vault]], [[neural-forge|Neural Forge]], [[neural-lens|Neural Lens]].

## Functional Requirements

- [ ] Ingest files from watched folders and detect create/update/delete events.
- [ ] Store original files or references with checksums, metadata, and provenance.
- [ ] Parse common documents: PDF, DOCX, PPTX, XLSX, HTML, Markdown, text, code.
- [ ] Parse media: images, audio, video via OCR/captioning/transcription/keyframe extraction.
- [ ] Generate embeddings and store chunks in a vector DB.
- [ ] Extract entities/relationships and store them in a graph DB.
- [ ] Resolve extracted mentions and source-system records into canonical real-world entity instances with aliases, identifiers, confidence scores, and review state.
- [ ] Maintain a living ontology of entity types, relationship types, business concepts, lifecycle states, metrics, policies, and decision logic.
- [ ] Link ontology definitions and business rules to source evidence, graph entities, provenance, permissions, and dataset governance metadata.
- [ ] Provide human chat interface with citations.
- [ ] Provide semantic search with filters and source previews.
- [ ] Operate headlessly by default with external agents able to perform all major functions through structured APIs.
- [ ] Provide CLI and REST API for agents.
- [ ] Publish commands.json or equivalent machine-readable command manifest.
- [ ] Provide review/approval task APIs for entity resolution, ontology changes, business rules, dataset governance, exports, training jobs, and revocation workflows.
- [ ] Support Neural Vault as an agent memory/context engine for personal OpenClaw agents and small-business OpenClaw agents.
- [ ] Provide native persistent notes, tables, and structured records so agents can use Neural Vault directly in workflows instead of relying only on Obsidian/Notion/Markdown.
- [ ] Provide a Context Pack Builder that assembles task-specific context from memories, notes, tables, entities, rules, decisions, source citations, and recent events.
- [ ] Provide an optional visualization framework with a default Knowledge Explorer for graph, ontology, evidence, entity resolution, rule/decision logic, and dataset lineage inspection.
- [ ] Allow external agents to define scenario-specific views through declarative view specifications covering entities, relationships, KPIs, filters, visual primitives, and actions.
- [ ] Support external connectors beginning with S3-compatible storage and Microsoft Graph/SharePoint.
- [ ] Support local LLM/embedding workers.
- [ ] Maintain a data rights registry that separates retrieval permissions from training/export/model-use permissions.
- [ ] Track provenance from source documents through chunks, summaries, extracted entities, generated examples, frozen datasets, and trained models.
- [ ] Provide a governed dataset builder for curated train/validation/test splits, dataset cards, approvals, and audit manifests.
- [ ] Support redaction/anonymization workflows before generating training examples.
- [ ] Support human review queues for training eligibility, generated examples, dataset freeze, and fine-tune/export approvals.
- [ ] Support agent-mediated human-in-the-loop workflows where agents prepare evidence, humans decide, policy validates authority, and the system records audit trails.

## Non-Functional Requirements

- [ ] Self-hostable via Docker Compose.
- [ ] Cloud-deployable to Azure/AWS.
- [ ] Permission-aware retrieval.
- [ ] Durable indexing jobs with retries and audit logs.
- [ ] Observable ingestion pipeline: status, failures, queue depth, file coverage.
- [ ] Citation/provenance for answers.
- [ ] Extensible connector/plugin architecture.
- [ ] Deterministic policy enforcement for indexing, retrieval, summarization, training, export, and data-boundary decisions.
- [ ] Revocation workflow that identifies affected downstream artifacts, datasets, and models when source rights change.
- [ ] Dataset quality scoring for duplication, source diversity, OCR confidence, stale data, sensitivity, contradiction clusters, and train/eval contamination.
- [ ] Versioned, reviewable semantic layer so ontology concepts, entity types, relationship types, and decision rules can evolve without losing audit history.
- [ ] Agent-first review architecture: optional UI only; core review workflows must work through APIs, events, commands, and external messaging surfaces.
- [ ] Full technology stack should be used across hardware tiers for build/maintenance simplicity; avoid a separate Lite software architecture unless performance forces it.
- [ ] Visualization must be permission-aware, audit-backed, and implemented as a client of the headless APIs rather than an authoritative state store.

## V1 Candidate Scope

- First deployment target: combined personal OpenClaw memory appliance and small-business knowledge base.
- Watched local folders and OpenClaw workspace/project files.
- PDF/Office/text/image/audio support.
- PostgreSQL + MinIO/filesystem + Qdrant + Neo4j.
- Docling primary parser, Tika fallback.
- Local embeddings and transcription.
- Optional web UI: search + chat + source viewer.
- Agent API/CLI with search, retrieve, summarize, graph-neighborhood commands.
- Review/approval service with task queue, evidence packages, policy checks, audit logs, webhooks/events, and external messaging adapters.
- Visualization framework V1: graph neighborhood, ontology map, node/edge details, KPI cards, tables/charts, timeline, evidence panel, entity resolution panel, and review actions.
- Dataset governance foundation: rights registry, provenance graph, redaction pipeline, dataset builder, review workflow, and audit logs.
- Semantic layer foundation: canonical entity registry, entity resolution workflow, typed business ontology, rule/decision-logic registry, and semantic query API.
- Native agent notes/tables and context pack APIs: `create_note`, `update_note`, `search_notes`, `create_table`, `upsert_table_row`, `query_table`, `store_memory`, `propose_fact`, `promote_memory`, `build_context_pack`.

## Later Scope

- SharePoint/Microsoft Graph connector.
- Video scene understanding and OCR over frames.
- Full graph explorer UI.
- Advanced ontology management: RDF/OWL export, SHACL-like validation, formal rule engine integration, ontology marketplace/templates, and deeper industry-specific semantic packs.
- MCP server.
- NVIDIA Dynamo/DGX Spark optimized inference profile.
- Model Foundry / Neural Forge layer for governed fine-tuning, distillation, and specialist model training using frozen approved datasets.
