# Neural Forge

**Parent:** [[neural-foundry|Neural Foundry]]  
**Category:** governed dataset and model adaptation layer

## Definition

**Neural Forge** is the dataset and model adaptation layer within Neural Foundry.

It turns approved, rights-checked organizational knowledge into curated datasets, evaluation sets, fine-tuned models, distilled specialist models, and model registry artifacts.

## Scope

Neural Forge includes:

- data rights-aware dataset building
- dataset cards and audit manifests
- train/validation/test split management
- redaction and anonymization workflows
- synthetic data generation where allowed
- human review and dataset freeze approvals
- evaluation set management
- LoRA / QLoRA fine-tuning
- DPO / preference tuning
- distillation into smaller specialist models
- model registry and deployment status
- dependency tracking from model → dataset → source evidence
- revocation impact analysis

## Important boundary

Neural Forge should not pull directly from arbitrary raw documents or unconstrained retrieval results.

It should consume only:

```text
approved source scopes
  → governed artifacts
  → reviewed/redacted examples
  → frozen dataset versions
  → approved training/export jobs
```

Retrieval permission is not training permission.

## Relationship to other modules

- [[neural-foundry|Neural Foundry]] is the umbrella platform.
- [[neural-vault|Neural Vault]] provides the governed knowledge substrate and semantic artifacts.
- [[neural-lens|Neural Lens]] can visualize dataset lineage, eval results, and model dependencies.

## Key linked docs

- [[dataset-governance|Dataset Governance Pipeline]]
- [[model-strategy|Model Strategy]]
- [[semantic-layer-ontology|Semantic Layer, Living Ontology, and Business Logic]]
- [[security-permissions-model|Security and Permissions Model]]
- [[headless-agent-review|Headless Agent-First Review]]
- [[requirements|Requirements]]
- [[tech-requirements|Technology Requirements]]

## Product description

> Neural Forge is the governed model adaptation layer that converts approved enterprise knowledge into training datasets, evaluation packs, fine-tunes, and specialist models with full lineage and rights control.
