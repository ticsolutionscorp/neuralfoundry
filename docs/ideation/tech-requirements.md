# AI Knowledge NAS — Technology Requirements

## Product family

Technology requirements for [[neural-foundry|Neural Foundry]], including [[neural-vault|Neural Vault]], [[neural-forge|Neural Forge]], and [[neural-lens|Neural Lens]].

## Core Capabilities

- File/object storage with checksums, versions, metadata, and provenance.
- Continuous ingestion from local folders, SMB/NFS-style shares, S3-compatible buckets, SharePoint/Microsoft Graph, databases, and SaaS connectors.
- Multimodal parsing for PDFs, Office docs, text, code, images, audio, video, email, archives, and scanned documents.
- Local LLM/VLM/embedding workers for extraction, summarization, classification, entity/relation extraction, and image/audio/video understanding.
- Vector index for semantic retrieval.
- Graph database for extracted mentions, canonical entities, relationships, document lineage, source references, ontology-to-instance links, inferred relationships, and GraphRAG.
- Metadata database for permissions, jobs, file versions, source systems, chunks, audit logs, dataset governance, and model lineage.
- Data rights registry separating retrieval permissions from summarization, entity extraction, synthetic-data generation, training, export, and boundary-crossing rights.
- Provenance graph linking source documents to parsed artifacts, chunks, summaries, extracted mentions, canonical entities, ontology statements, business rules, training examples, dataset versions, and trained models.
- Entity resolution layer that links extracted mentions and source-system records to canonical real-world entity instances using deterministic IDs, fuzzy matching, embeddings, confidence scoring, source-priority rules, and human merge/split review.
- Living ontology / semantic layer for entity types, relationship types, business concepts, lifecycle states, events, KPIs, policies, rules, decision logic, evidence links, review status, and versioning.
- Business rules and decision-logic registry capturing approval paths, scoring logic, escalation rules, workflow rules, policy exceptions, operational heuristics, and source-backed explanations.
- Governed dataset builder for filtering, curation, deduplication, train/validation/test splits, frozen dataset versions, dataset cards, and audit manifests.
- Redaction/anonymization pipeline for PII, secrets, client names, sensitive entities, and policy-based transformations before training-example generation.
- Headless agent-first operation model where external agents can perform all major functions through structured APIs, commands, events, and workflow integrations.
- Human-in-the-loop review service for training eligibility, generated example approval, dataset freeze, fine-tune/export approval, entity merge/split review, ontology/rule approvals, and revocation handling.
- Review task queue with evidence packages, proposed actions, confidence/risk scoring, required reviewer roles, policy validation, decisions, comments, and audit logs.
- Optional Human UI: chat, semantic search, filters, source viewer, graph explorer, ontology explorer, entity review/merge UI, rule review UI, visualization framework, default Knowledge Explorer, admin dashboard. UI should consume the same APIs as agents and remain optional.
- Declarative visualization framework that lets external agents create scenario-specific views using controlled view specifications rather than arbitrary frontend code.
- Visualization API for saved views, graph expansion, evidence panels, KPI dashboards, maps/charts/timelines, review actions, and permission-aware rendering.
- Agent UI: CLI, REST API, commands.json, webhooks/events, possibly MCP server/skill, including semantic query, review, and visualization commands such as `resolve_entity`, `get_entity`, `explain_concept`, `explain_rule`, `trace_evidence`, `find_rules_applicable_to`, `list_review_tasks`, `explain_review_task`, `approve_review_task`, `create_view`, `validate_view_spec`, and `expand_view_node`.
- Deployment: Docker Compose for V1, Kubernetes later, self-hosted or Azure/AWS.

## Candidate Open-Source Components

### Parsing / Document Intelligence

- Docling: strong candidate for structured document parsing, layout, tables, OCR, multimodal RAG preparation.
- Apache Tika: broad file type detection/extraction, useful as fallback for many formats.
- Unstructured: mature RAG ingestion library, broad connector/document support.
- FFmpeg: media extraction/transcoding.
- Whisper/faster-whisper: audio transcription.
- OCR: Tesseract, PaddleOCR, or Docling-integrated OCR paths.

### Storage / Databases

- PostgreSQL: metadata, jobs, permissions, relational state, data rights registry, ontology/rule registry, canonical entity registry, dataset registry, model registry, approvals, and audit logs.
- MinIO or filesystem/S3-compatible object storage: authoritative object/file store.
- Qdrant: lean self-hostable vector DB.
- Weaviate/Milvus: alternatives if scale/module needs differ.
- Neo4j: mature graph DB and GraphRAG ecosystem; also useful for canonical entity graphs, ontology-to-instance links, provenance lineage, inferred relationships, rule evidence, and model/dataset dependency tracking.
- ArangoDB: possible single multi-model alternative if reducing database count is a priority.
- Optional semantic-web compatibility later: RDF/OWL export, SHACL-style validation, or an RDF triple store if customer requirements demand formal ontology standards.

### AI / Retrieval Framework

- LlamaIndex: strong for indexing, knowledge graph/property graph, retrieval patterns; useful for candidate ontology/relationship extraction workflows but not as the source of truth.
- LangChain/LangGraph: broad agent workflow ecosystem.
- Haystack: production RAG pipelines and search workflows.
- Local inference: Ollama, vLLM, SGLang, TensorRT-LLM, optionally NVIDIA Dynamo for GPU inference orchestration.
- Embeddings/reranking: local BGE/E5/Nomic/Jina-style embedding models; rerankers for retrieval quality.

### Visualization

- Next.js / React: optional Knowledge Explorer and custom view client.
- Sigma.js or Cytoscape.js: interactive graph visualization.
- React Flow: ontology maps, rule flows, and directed decision-logic diagrams.
- Visualization API over Neo4j/Postgres/Qdrant with server-side permission checks, query limits, and graph expansion controls.
- Optional comparison/internal tools: Neo4j Bloom, Graphistry, Linkurious, Kùzu Explorer, depending on customer/deployment needs.

### Queues / Orchestration

- Temporal, Prefect, Celery, BullMQ, or n8n for ingestion jobs, review workflows, durable waits, approval routing, retries, and event-driven coordination.
- Event watcher: fswatch/inotify/FSEvents for local; webhook/polling for SaaS/cloud sources.
- Webhook/event bus for review events such as `review_task.created`, `review_task.approved`, `review_task.rejected`, `review_task.escalated`, and `review_task.needs_more_evidence`.

### Connectors

- Microsoft Graph / SharePoint / OneDrive.
- AWS S3 / S3-compatible stores.
- Google Drive / Workspace.
- Slack/Teams/email/Telegram for notifications, approvals, and agent-mediated review workflows.
- SQL databases via read-only connectors.

## Deployment Targets

- Self-hosted Docker on AI-capable workstation/server.
- DGX Spark / OSFoundry profile with local inference acceleration.
- Cloud profile for Azure/AWS with managed object storage, GPU workers optional.

## Security / Compliance Requirements

- Preserve source permissions where possible.
- Verify reviewer identity and authority before accepting approval/rejection decisions from agents or external messaging channels.
- Per-user and per-agent authorization checks at retrieval time.
- Separate source access permissions from data-use rights such as `can_train`, `can_export`, and `can_leave_boundary`.
- Deterministic policy engine for rights enforcement; LLMs must never decide whether data can be used.
- Revocation/unlearning workflow: block future use, identify affected artifacts/datasets/models, quarantine or retire impacted models, and produce audit reports.
- Audit logs for indexing, reads, exports, agent queries.
- Provenance/citations for every answer.
- Encryption at rest and in transit.
- Secrets via Docker secrets/Vault-style integration.
