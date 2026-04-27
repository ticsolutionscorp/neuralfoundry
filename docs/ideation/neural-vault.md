# Neural Vault

**Parent:** [[neural-foundry|Neural Foundry]]  
**Category:** local/self-hosted AI knowledge NAS and semantic knowledge appliance

## Definition

**Neural Vault** is the local/self-hosted AI-native knowledge appliance within Neural Foundry.

It is the secure, permission-aware knowledge substrate: storage, parsing, indexing, graph, ontology, entity resolution, review workflows, agent APIs, native notes/tables, and task-specific context packs.

The V1 target combines a personal OpenClaw memory appliance and small-business knowledge base. Neural Vault should make one or two OpenClaw agents smarter by giving them persistent structured memory, source-backed recall, semantic context, and durable workflow state.

## Scope

Neural Vault includes:

- file/object storage and source references
- watched folders and external connectors
- Docling/Tika/Unstructured-style parsing
- OCR, transcription, and multimodal extraction
- metadata store
- vector index
- graph database
- canonical entity registry
- entity resolution
- living ontology and semantic layer
- business rules / decision logic registry
- permissions and data rights registry
- provenance and lineage
- headless agent APIs
- native persistent notes/tables and structured records
- context pack builder
- review/approval service

Neural Vault does **not** primarily mean secrets/password storage. OpenClaw may use Vault for secrets, but Neural Vault is the product name for the AI knowledge appliance.

## Relationship to other modules

- [[neural-foundry|Neural Foundry]] is the umbrella platform.
- [[neural-lens|Neural Lens]] provides visual inspection and custom scenario views over Neural Vault.
- [[neural-forge|Neural Forge]] consumes governed datasets and semantic artifacts from Neural Vault for model adaptation.

## Key linked docs

- [[requirements|Requirements]]
- [[tech-requirements|Technology Requirements]]
- [[storage-backend-comparison|Storage Backend Comparison]]
- [[hardware-requirements|Hardware Requirements]]
- [[security-permissions-model|Security and Permissions Model]]
- [[semantic-layer-ontology|Semantic Layer, Living Ontology, and Business Logic]]
- [[headless-agent-review|Headless Agent-First Review]]
- [[model-strategy|Model Strategy]]
- [[agent-memory-context-engine|Agent Memory and Context Engine]]

## Product description

> Neural Vault is a self-hosted AI knowledge NAS that converts organizational files, databases, SaaS records, and media into a permission-aware, agent-accessible semantic knowledge graph.
