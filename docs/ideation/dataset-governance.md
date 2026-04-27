# AI Knowledge NAS — Dataset Governance Pipeline

## Product family

Defines governance foundations for [[neural-forge|Neural Forge]], using knowledge from [[neural-vault|Neural Vault]]. Parent: [[neural-foundry|Neural Foundry]].

**Date:** 2026-04-26
**Status:** Ideation / architecture direction

## Why this matters

If the AI Knowledge NAS eventually supports fine-tuning open-weight models, distilling smaller industry-specific models, or exporting curated training datasets, it needs a governed dataset pipeline before any Model Foundry / Neural Forge layer is added.

The key principle: **retrieval permission is not training permission.** A user or agent may be allowed to read a document, but that does not automatically mean the document can be used to train, fine-tune, export, or contribute to shared models.

The dataset governance pipeline becomes a major product moat: enterprises need to know exactly why a training example was allowed into a dataset, where it came from, who approved it, and what models depend on it.

## Required governance modules

### 1. Data rights registry

Track rights independently from normal document access permissions.

Per source, document, chunk, and derived artifact, capture flags such as:

- `can_index`
- `can_retrieve`
- `can_summarize`
- `can_extract_entities`
- `can_generate_synthetic_data`
- `can_train`
- `can_export`
- `can_leave_boundary`
- retention and expiry rules
- license / contractual restrictions
- data owner / steward
- client / tenant boundary
- approved model-use scope: internal only, client-specific, industry model, shared/global model

### 2. Provenance graph

Every derived artifact must have lineage back to source material.

Track:

- source connector and source system ID
- source file/document version and checksum/hash
- parser/OCR/VLM model and version
- extraction/summarization prompt, model, and version
- chunk IDs and source spans
- generated training examples
- human reviewers and approvals
- dataset version membership
- fine-tuned models derived from the dataset

This lineage enables revocation, audit reports, dataset cards, and model dependency tracking.

### 3. Dataset builder / curation layer

Datasets should be created like controlled releases, not ad-hoc exports.

Capabilities:

- filter by tenant, client, industry, collection, document type, sensitivity, date, owner, and rights flags
- include/exclude document types and source systems
- deduplicate near-identical content
- balance topics, classes, source diversity, and task types
- split train / validation / test sets
- freeze immutable dataset versions
- generate dataset cards and audit manifests
- prevent eval data from leaking into training data

### 4. Redaction and transformation pipeline

Before source material becomes training data, the system needs policy-controlled transformation.

Capabilities:

- PII detection and masking
- secrets detection
- client/company/person anonymization
- synthetic replacement where allowed
- reversible vs irreversible redaction policies
- source-span-aware redaction so citations and lineage survive
- human approval for high-risk transformations

Redaction should happen **before** generated Q&A / instruction examples are produced, not after.

### 5. Human review workflow

Enterprise-grade training data needs explicit review gates.

Review queues:

- approve source collection for training eligibility
- approve generated Q&A / instruction examples
- reject hallucinated, low-quality, stale, or sensitive examples
- mark examples as sensitive or client-specific
- approve dataset freeze
- approve export or fine-tune job

V1 can be lightweight, but the workflow must exist.

### 6. Dataset quality scoring

Before training, score datasets for risk and usefulness.

Checks:

- duplication / near-duplication
- source diversity
- class or topic imbalance
- low-confidence OCR / transcription
- stale source documents
- contradiction clusters
- toxic/sensitive content
- unsupported claims
- excessive synthetic content
- train/eval contamination

### 7. Evaluation set management

Evaluation data must be managed separately from training data.

Support:

- golden Q&A sets
- task-specific benchmarks
- industry-specific eval packs
- regression tests
- safety/privacy evals
- citation-grounding checks
- refusal tests for restricted or insufficient-evidence questions
- baseline comparison against the base model and previous model versions

### 8. Dataset and model registry

Track datasets and model artifacts as first-class governed objects.

Dataset registry fields:

- dataset ID and version
- dataset card
- source scope and rights summary
- approval status
- train/validation/test split hashes
- redaction policy used
- generation models/prompts used
- quality scores

Model registry fields:

- model ID and version
- base model
- adaptation method: RAG-only, LoRA/QLoRA, DPO/preference tuning, continued pretraining, distillation, full fine-tune
- training dataset versions
- evaluation dataset versions
- training config
- eval results
- deployment status
- rollback target
- revocation/dependency status

### 9. Policy engine

The system needs deterministic policy enforcement. The LLM must never decide whether data can be used.

Example policies:

- Client data can never train shared/global models.
- Only anonymized support tickets may train industry models.
- Legal documents can be retrieved but not exported or used for training.
- Internal docs can train internal models only.
- Anything marked confidential requires owner approval before dataset inclusion.
- No artifact may leave the appliance unless `can_leave_boundary = true`.

Implementation can start in Postgres/app logic, then evolve toward OPA/Cedar-style policy evaluation if needed.

### 10. Revocation / unlearning workflow

True machine unlearning is hard, but operational revocation is mandatory.

Required workflow:

- mark source document/collection revoked
- block future dataset inclusion
- identify affected chunks, examples, datasets, and models through the provenance graph
- quarantine or retire affected datasets/models
- retrain from clean dataset when required
- produce audit report for compliance/customer review

## Recommended governed pipeline

```text
Source ingestion
  → rights classification
  → parsing/enrichment
  → governed artifact store
  → redaction/transformation
  → curated dataset draft
  → human review
  → frozen dataset version
  → evaluation pack selection
  → training/export approval
  → model registry / deployment
```

## Design principles

1. **Separate retrieval rights from training rights.**
2. **Attach permissions and lineage to every derived artifact.**
3. **Use deterministic policy checks, not LLM judgment.**
4. **Keep eval data separate from training data.**
5. **Make dataset creation auditable and repeatable.**
6. **Support revocation even if true model unlearning is not feasible.**
7. **Treat dataset governance as a core product layer, not an afterthought.**

## Relationship to future Model Foundry / Neural Forge layer

The Model Foundry / Neural Forge layer should only consume frozen, approved, governed datasets. It should not reach directly into raw documents or arbitrary retrieval results.

This keeps the training layer clean:

```text
AI Knowledge NAS / governed knowledge layer
  → approved dataset versions
  → Model Foundry / Neural Forge training jobs
  → model registry
  → deployed specialist models
```

Potential model adaptation modes:

- RAG-only: safest default, no model weights changed.
- LoRA / QLoRA: style, terminology, workflows, classification, form-filling.
- DPO / preference tuning: human correction and answer preference alignment.
- Continued pretraining: deeper domain language adaptation; expensive and higher-risk.
- Distillation / small specialist models: compact industry/task-specific models.
