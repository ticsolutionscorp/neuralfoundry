# AI Knowledge NAS — Project Ideation

## Product family

Part of [[neural-foundry|Neural Foundry]]. Related modules: [[neural-vault|Neural Vault]], [[neural-forge|Neural Forge]], [[neural-lens|Neural Lens]].

## Problem Statement

Organizations and AI agents accumulate files, documents, media, databases, SaaS records, and operational knowledge across fragmented systems. Existing NAS/file storage preserves bytes but does not understand content. Existing RAG tools often index narrow document sets and do not provide durable storage, graph relationships, agent-native APIs, or full lifecycle crawling.

The project direction is an AI-native knowledge base / NAS that combines object/file storage, relational metadata, vector search, graph knowledge, multimodal file understanding, local LLM extraction, continuous indexing, human chat/search UI, and agentic CLI/API access.

## Solution Direction

Build a self-hostable and cloud-deployable "AI NAS" that watches files and connected data sources, extracts structure and meaning, indexes into vector + graph + metadata stores, and exposes knowledge through human and agent interfaces.

## Initial Product Names / Concepts

- KnowledgeNAS
- Neural Vault
- AgentVault
- Cognition NAS
- DataFoundry
- Membrane

## Key Questions to Explore

- Is the primary buyer/use case enterprise knowledge management, AI agent infrastructure, compliance/eDiscovery, or personal/team NAS?
- Should storage be authoritative, or should the system index existing storage locations without taking ownership of files?
- What is the minimum viable file type coverage for V1?
- Should the first deployment target DGX Spark/on-prem hardware, generic Docker, or cloud?
- How much local LLM capability is required vs. optional cloud model enhancement?
- Should graph extraction be automatic for all documents or selectively applied to high-value corpora?

## Session Notes

### 2026-04-25 — Initial concept from Nick

Nick wants a smart knowledge base combining storage, database, vector database, graph, file handling for documents/videos/audio/images, local LLM extraction, indexing/crawling as files change, human interface via chat/semantic search, and agentic interface via CLI/API. It should work like an AI NAS with strong built-in capabilities and extensible connectors for third-party databases/SaaS such as Microsoft Graph/SharePoint and AWS S3. Deployable self-hosted on AI-capable hardware or in Azure/AWS. Should combine open-source components rather than reinventing every layer.

### 2026-04-26 — Dataset governance before model training

Nick wants to explore future use of the system for fine-tuning open-weight models and eventually training smaller industry-specific models. Architecture direction: before adding a Model Foundry / Neural Forge layer, the knowledge system needs a dataset governance pipeline. Retrieval permissions are not training permissions. The system must track data rights, provenance, redaction, human approvals, dataset versions, eval separation, model lineage, policy enforcement, and revocation workflows. Captured in `dataset-governance.md`.

### 2026-04-26 — Living ontology and business decision logic

Nick confirmed a living ontology is required. The semantic layer should model business context beyond raw retrieval: entity types, canonical entities, relationships, business concepts, lifecycle states, KPIs, rules, policies, and decision logic. It should explain not just what data exists, but why decisions are made. Entity resolution is a prerequisite: extracted mentions and source records must resolve to unique real-world entity instances before the ontology and graph become reliable. Captured in `semantic-layer-ontology.md`.

### 2026-04-26 — Headless agent-first review model

Nick wants the usage model to be headless, where external agents interact with the system for all functions, including human-in-the-loop review and approval workflows. Direction: human review should be a governed review/approval service with APIs, task queues, evidence packages, policy checks, audit logs, events, and external messaging adapters. A web UI can exist later, but should be optional and consume the same APIs as agents. Captured in `headless-agent-review.md`.

### 2026-04-26 — Visualization framework and Knowledge Explorer

Nick wants humans to be able to visually inspect the ontology/graph/knowledge system even though the core is headless. Direction: create a separate visualization framework with a default Knowledge Explorer, plus agent-configurable declarative views for scenarios such as customer risk, entity resolution, ontology/rule inspection, KPI dashboards, maps, charts, timelines, and dataset/model lineage. The visual layer is optional, permission-aware, audit-backed, and consumes the same headless APIs as agents. Captured in `visualization-framework.md`.

### 2026-04-26 — Project creation decisions

Nick confirmed the project should be created as one OSForge project named Neural Foundry, with Neural Vault, Neural Lens, and Neural Forge as modules. V1 target combines personal OpenClaw memory appliance and small-business knowledge base. V1 scope: Neural Vault first, minimal Neural Lens, Neural Forge as architecture placeholder. Initial connectors: local folders, Obsidian-like note folders when present, OpenClaw workspace/project files, plus Neural Vault-native notes/tables. Keep the full technology stack across hardware tiers for build/maintenance simplicity; no separate Lite software profile. Repo/agent slug: `neuralfoundry`. Captured in `agent-memory-context-engine.md`.

## Early Architecture Hypothesis

Layered architecture:

1. Storage layer: file/object storage + metadata DB.
2. Ingestion layer: watchers, crawlers, connectors, change detection.
3. Parsing layer: Docling/Tika/Unstructured-style extraction, OCR, ASR, image/video captioning.
4. Knowledge layer: relational metadata + vector DB + graph DB.
5. Intelligence layer: local LLM/VLM/embedding/reranker workers.
6. Retrieval layer: hybrid RAG + GraphRAG + metadata filters + provenance.
7. Interfaces: web UI, chat, semantic search, CLI, REST API, commands manifest, MCP/agent skill.
8. Operations layer: queues, job orchestration, observability, permissions, audit logs.
