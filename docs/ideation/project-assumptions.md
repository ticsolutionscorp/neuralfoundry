# AI Knowledge NAS — Assumptions to Validate

## Product family

Assumptions for [[neural-foundry|Neural Foundry]] and related modules: [[neural-vault|Neural Vault]], [[neural-forge|Neural Forge]], [[neural-lens|Neural Lens]].

## Product Assumptions

- Users want one unified system that stores and understands data, not just another RAG app pointed at a folder.
- Agentic CLI/API access is a differentiator, not just a nice-to-have.
- Local/private AI processing is important enough to justify more deployment complexity.
- Enterprises will accept a system that combines several open-source infrastructure components if deployment/admin is clean.

## Technical Assumptions

- Docling can handle enough of the target document workload to be the primary parser.
- Tika/Unstructured can cover long-tail file formats not handled well by Docling.
- A hybrid PostgreSQL + object storage + vector DB + graph DB design is manageable if wrapped behind a clean API.
- Local embedding and extraction models are sufficient for most indexing; cloud/frontier models can be optional enrichment.
- Permissions can be preserved or approximated across connected systems like SharePoint/S3/databases.

## Risk Areas

- Scope explosion: NAS + RAG + graph + connectors + UI + agent API is large.
- Permissions are hard; answering from data the user should not see is unacceptable.
- Multimodal indexing can become expensive and slow at scale.
- Graph extraction quality may be uneven without domain-specific schemas.
- Connectors can dominate engineering time.

## Validation Questions

- Who is V1 for: TIC internal use, OSFoundry clients, or a standalone commercial product?
- Should V1 own storage or index existing storage only?
- What are the first 3 source types that matter most?
- What scale should V1 handle: 100GB, 1TB, 10TB, 100TB?
- Is graph exploration a visible user feature in V1 or primarily an internal retrieval mechanism?
- What local hardware profile is the default: CPU-only, single GPU workstation, DGX Spark?
