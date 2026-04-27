# Neural Foundry Agent Briefing

## Purpose

Neural Foundry is the umbrella platform for AI-native organizational knowledge, memory, semantics, visualization, and governed model adaptation.

The V1 target combines:

1. A personal OpenClaw memory appliance.
2. A small-business knowledge base.

The goal is to make OpenClaw agents, or similar durable AI workers, materially smarter by giving them source-backed recall, structured notes/tables, memory scopes, semantic context, entity resolution, and task-specific context packs.

## Key Architecture Split

- **AgentOS** manages who agents are: identity, registry, roles, capabilities, communication, status, and discovery.
- **Neural Vault** manages what agents know: memory stores, source-backed knowledge, context retrieval, graph, ontology, rules, permissions, and context pack generation.

Short version:

> AgentOS tells us who the agent is and what it can do. Neural Vault tells the agent what it needs to know.

## Documentation to Read First

1. `docs/ideation/neural-foundry.md`
2. `docs/ideation/agent-memory-context-engine.md`
3. `docs/ideation/neural-vault.md`
4. `docs/ideation/requirements.md`
5. `docs/ideation/tech-requirements.md`
6. `docs/ideation/security-permissions-model.md`
7. `docs/ideation/semantic-layer-ontology.md`
8. `docs/ideation/model-strategy.md`
9. `docs/ideation/hardware-requirements.md`

## Current Design Work

The next deliverable is a formal design specification for Nick approval. Do not start build implementation until the design is approved.

Important open questions are captured in `CONTEXT.md` in the agent workspace and should be refined through Slack design discussion.
