# Neural Lens

**Parent:** [[neural-foundry|Neural Foundry]]  
**Category:** visualization framework and Knowledge Explorer

## Definition

**Neural Lens** is the optional visualization and inspection layer within Neural Foundry.

It provides a default Knowledge Explorer plus a declarative framework for agents to create custom scenario-specific views over the ontology, graph, entities, KPIs, evidence, review tasks, datasets, and models.

## Scope

Neural Lens includes:

- default Knowledge Explorer
- ontology map
- entity instance graph
- evidence/provenance panels
- entity resolution review views
- rule / decision logic views
- dataset and model lineage views
- KPI cards and dashboards
- charts, timelines, maps, and graph neighborhoods
- declarative view specifications
- saved views
- permission-aware rendering
- review actions from visual contexts

## Design boundary

Neural Lens is not the authority or state owner.

It is a client of the headless core:

```text
Neural Vault / Neural Foundry APIs
  → Visualization API
  → Neural Lens views
```

Agents, CLI clients, and Neural Lens should all use the same underlying APIs, permissions, commands, and review workflows.

## Relationship to other modules

- [[neural-foundry|Neural Foundry]] is the umbrella platform.
- [[neural-vault|Neural Vault]] provides the graph, ontology, entity registry, evidence, and review tasks that Neural Lens visualizes.
- [[neural-forge|Neural Forge]] provides datasets, eval sets, model lineage, and training results that Neural Lens can inspect.

## Key linked docs

- [[visualization-framework|Visualization Framework and Knowledge Explorer]]
- [[semantic-layer-ontology|Semantic Layer, Living Ontology, and Business Logic]]
- [[headless-agent-review|Headless Agent-First Review]]
- [[dataset-governance|Dataset Governance Pipeline]]
- [[requirements|Requirements]]
- [[tech-requirements|Technology Requirements]]

## Product description

> Neural Lens is the visual trust layer for Neural Foundry: an optional, permission-aware Knowledge Explorer and agent-configurable visualization framework for inspecting the business graph, ontology, evidence, KPIs, and model lineage.
