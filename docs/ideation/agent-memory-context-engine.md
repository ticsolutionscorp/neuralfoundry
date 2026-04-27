# Neural Foundry — Agent Memory and Context Engine

## Product family

Part of [[neural-foundry|Neural Foundry]]. Core capability of [[neural-vault|Neural Vault]], optionally visualized by [[neural-lens|Neural Lens]] and later used by [[neural-forge|Neural Forge]].

**Date:** 2026-04-26
**Status:** Project-creation decision / V1 target

## Core decision

The first deployment target should combine:

1. **Personal OpenClaw memory appliance**
2. **Small-business knowledge base**

This is the V1 target for first deployment.

Neural Vault should be capable enough to support a personal OpenClaw agent or a small-business OpenClaw agent as its knowledge base, memory substrate, and context engine.

## Relationship to Obsidian

Agents may continue using Obsidian or a similar local notes system as their per-agent notepad / working memory.

Neural Foundry should not depend on Obsidian as part of the product. The project can assume agents may already have Obsidian or an equivalent local note workspace, but Neural Vault should provide the canonical shared knowledge/context layer.

Recommended model:

```text
Per-agent notepad / Obsidian-like workspace
  - scratchpad
  - daily notes
  - temporary task state
  - local working docs
  - agent-specific observations

Neural Vault
  - canonical/shared knowledge
  - source-backed facts
  - entity graph
  - ontology
  - business rules / decision logic
  - long-term structured notes and tables
  - context packs
  - permission-aware retrieval
```

Agents can operate without Neural Vault. When Neural Vault and related plugins are installed, agents become smarter and more capable through richer memory, context retrieval, shared knowledge, and structured state.

## Agent-native notes and tables

Neural Vault should support native persistent notes, tables, and structured records so agents can use the database directly as part of their workflow.

This means agents should not be forced to store every durable process artifact in Obsidian, Notion, Markdown, or external notes.

Examples:

- project notes
- decision logs
- task state tables
- recurring process checklists
- entity-specific notes
- customer/vendor/account notes
- structured operating tables
- workflow state
- review queues
- context annotations

Agent-facing commands should include:

- `create_note`
- `update_note`
- `search_notes`
- `link_note_to_entity`
- `create_table`
- `upsert_table_row`
- `query_table`
- `link_table_row_to_entity`
- `store_memory`
- `propose_fact`
- `promote_memory`
- `build_context_pack`

## Multi-agent model

For multi-agent and enterprise deployments, AgentOS and Neural Vault should have separate responsibilities.

### AgentOS

AgentOS manages:

- agent identity
- agent registry
- agent roles/scopes
- agent capabilities
- project assignment
- agent discovery
- agent communication/status

### Neural Vault

Neural Vault manages:

- agent memory stores
- shared knowledge
- context retrieval
- semantic graph
- entity resolution
- ontology/rules
- source-backed knowledge
- memory promotion/review
- context pack generation
- permission-aware retrieval

Integration rule:

> AgentOS tells us who the agent is and what it can do. Neural Vault tells the agent what it needs to know.

## Memory scopes

Neural Vault should support memory scopes:

- global/org memory
- project memory
- agent memory
- user memory
- session/episodic memory
- shared team memory

Memory state should prevent pollution:

- ephemeral
- proposed
- reviewed
- canonical
- deprecated
- revoked

Agents can write observations freely to their working memory or proposed memory, but shared/canonical knowledge should require evidence, confidence, policy, or review.

## Context Pack Builder

A core V1 API should be a context pack builder.

Example:

```json
{
  "agent_id": "agentcms",
  "task": "Plan the next build step for authentication",
  "project_id": "AgentCMS",
  "user_id": "nick",
  "max_tokens": 12000,
  "include_recent_events": true,
  "include_entities": true,
  "include_decisions": true,
  "include_rules": true,
  "include_source_citations": true
}
```

Returned context should include:

- relevant facts
- project state
- entity summaries
- prior decisions
- user preferences
- applicable rules
- unresolved review tasks
- relevant notes/tables
- source citations
- confidence and recency markers

## V1 module scope

V1 should focus on:

- [[neural-vault|Neural Vault]] as the agent memory/context/knowledge engine
- minimal [[neural-lens|Neural Lens]] for visual inspection and trust
- [[neural-forge|Neural Forge]] as an architecture placeholder, not full V1 implementation

## V1 connectors

Initial connectors:

- local folders
- Obsidian or Obsidian-like note folders, when present
- OpenClaw workspace/project files
- Neural Vault native notes/tables

Later connectors:

- Google Drive / Workspace
- Microsoft Graph / SharePoint / OneDrive
- S3-compatible storage
- Slack/Teams/email
- SQL databases

## Hardware target

V1 should target the existing hardware range in [[hardware-requirements|Hardware Requirements]].

The low-end supported range starts at Personal Neural Vault Lite, but the product should use the full technology stack across versions for easier build and maintenance. Avoid a separate Lite software architecture if possible.

## Repository/project naming

Recommended project creation values:

- Project: Neural Foundry
- Repo slug: `neuralfoundry`
- Agent slug: `neuralfoundry`
- GitHub org: `ticsolutionscorp`
- Model: `zai/glm-5.1`
