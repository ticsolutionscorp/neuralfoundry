# Neural Foundry

**Status:** naming structure / parent product concept  
**Category:** umbrella platform

## Definition

**Neural Foundry** is the umbrella platform for AI-native organizational knowledge, semantics, governance, visualization, and model adaptation.

It contains the full product family:

- [[neural-vault|Neural Vault]] — local/self-hosted AI knowledge NAS and semantic knowledge appliance.
- [[neural-forge|Neural Forge]] — governed dataset and model adaptation layer.
- [[neural-lens|Neural Lens]] — optional visualization framework and Knowledge Explorer.

## Positioning

Neural Foundry turns fragmented organizational data into a governed, explainable, agent-accessible business knowledge model.

The first deployment target combines a personal OpenClaw memory appliance and small-business knowledge base: Neural Vault should make one or two agents significantly smarter by providing shared memory, durable notes/tables, context packs, entity-aware recall, and source-backed knowledge.

It combines:

- storage and ingestion
- document/media parsing
- vector retrieval
- graph knowledge
- entity resolution
- living ontology
- business rules / decision logic
- permissions and data rights governance
- headless agent APIs
- human-in-the-loop review workflows
- visual inspection surfaces
- governed datasets and future specialist model training

## Product architecture

```text
Neural Foundry
  ├─ Neural Vault
  │   ├─ Knowledge Core
  │   ├─ Semantic Layer
  │   ├─ Entity Resolution
  │   ├─ Permissions / Data Rights
  │   ├─ Review / Approval Service
  │   └─ Agent APIs
  │
  ├─ Neural Lens
  │   ├─ Knowledge Explorer
  │   ├─ Ontology / Graph Explorer
  │   ├─ KPI / Scenario Views
  │   └─ Agent-configurable Visualization Framework
  │
  └─ Neural Forge
      ├─ Dataset Governance
      ├─ Dataset Builder
      ├─ Evaluation Sets
      ├─ Fine-tuning / Distillation
      └─ Model Registry
```

## Core design principles

1. **Headless first.** Agents can operate all major functions through APIs, commands, events, and workflows.
2. **Human review is structured.** Humans participate through review tasks, approvals, evidence packages, and optional visual surfaces.
3. **Governed knowledge before training.** Model adaptation consumes frozen, approved, rights-checked datasets only.
4. **Living semantics.** The system builds and maintains a versioned, evidence-backed ontology of how the business works.
5. **Permission-aware by default.** Retrieval rights, training rights, export rights, and boundary-crossing rights are separate.
6. **Visual trust layer.** The core is headless, but humans must be able to inspect and challenge what the system is building.

## Linked notes

### Product modules

- [[neural-vault|Neural Vault]]
- [[neural-forge|Neural Forge]]
- [[neural-lens|Neural Lens]]

### Core documentation

- [[README|AI Knowledge NAS Ideation Index]]
- [[agent-memory-context-engine|Agent Memory and Context Engine]]
- [[ideation-notes|Ideation Notes]]
- [[business-context|Business Context]]
- [[requirements|Requirements]]
- [[tech-requirements|Technology Requirements]]
- [[project-assumptions|Project Assumptions]]
- [[research-notes|Research Notes]]

### Neural Vault / knowledge appliance docs

- [[storage-backend-comparison|Storage Backend Comparison]]
- [[hardware-requirements|Hardware Requirements]]
- [[security-permissions-model|Security and Permissions Model]]
- [[semantic-layer-ontology|Semantic Layer, Living Ontology, and Business Logic]]
- [[headless-agent-review|Headless Agent-First Review]]

### Neural Lens / visualization docs

- [[visualization-framework|Visualization Framework and Knowledge Explorer]]

### Neural Forge / training docs

- [[dataset-governance|Dataset Governance Pipeline]]
- [[model-strategy|Model Strategy]]
