# AI Knowledge NAS — Security and Permissions Model

## Product family

Supports [[neural-vault|Neural Vault]] and [[neural-forge|Neural Forge]], part of [[neural-foundry|Neural Foundry]].

## Core principle

The appliance must be permission-aware at retrieval time, not just at UI time. Every search, graph traversal, citation lookup, chunk read, document preview, and generated answer must be scoped to the requesting principal.

A principal can be:

- Human user.
- AI agent.
- Service account.
- External automation.
- Tenant/org.

The system should never retrieve unauthorized chunks into model context. Filtering after generation is not sufficient.

## Identity model

Use a central identity layer that maps every request to a canonical principal.

Canonical principal record:

```json
{
  "principal_id": "user_or_agent_uuid",
  "principal_type": "human|agent|service_account",
  "tenant_id": "org_uuid",
  "external_identities": [
    { "provider": "entra_id", "id": "...", "email": "..." },
    { "provider": "google_workspace", "id": "...", "email": "..." },
    { "provider": "slack", "id": "..." }
  ],
  "groups": ["group_uuid"],
  "roles": ["role_name"],
  "attributes": {
    "department": "finance",
    "clearance": "confidential"
  }
}
```

Identity providers:

- Microsoft Entra ID / Azure AD via OIDC/SAML + Microsoft Graph.
- Google Workspace via OIDC/SAML + Admin SDK.
- Okta/Auth0/Ping via OIDC/SAML/SCIM.
- Local Supabase Auth for standalone/small deployments.
- API keys/JWT client credentials for agents and service accounts.

## Authorization model

Use layered authorization:

1. Tenant boundary.
2. Source boundary.
3. Document ACL.
4. Chunk/artifact ACL.
5. Field-level or sensitivity labels where needed.
6. Runtime policy checks for answer generation and tool calls.

Do not rely only on RBAC. Use ACL + group membership + ABAC/policy rules.

Recommended policy stack:

- Postgres/Supabase RLS for metadata tables.
- OpenFGA or OPA/Cedar-style policy engine for relationship/attribute authorization.
- Source-specific ACL sync for SharePoint/Drive/SaaS systems.
- Signed service tokens/JWTs for agents.

## Permission-aware indexing

For each source connector, ingest both content and permissions.

### Local appliance files

For files stored directly on the appliance:

- Store ownership, ACLs, group memberships, POSIX/NFS/SMB permissions where applicable.
- Normalize them into canonical ACL records.
- Preserve source ACL provenance.
- Re-evaluate permissions on change events.

Example document ACL record:

```json
{
  "resource_id": "doc_uuid",
  "resource_type": "document",
  "tenant_id": "org_uuid",
  "allow_principals": ["user:...", "agent:..."],
  "allow_groups": ["group:finance"],
  "deny_principals": [],
  "deny_groups": [],
  "inherited_from": "folder_uuid",
  "source_acl_hash": "...",
  "last_synced_at": "..."
}
```

### External systems

For Microsoft Graph / SharePoint / OneDrive:

- Use Microsoft Graph permissions APIs to capture file/folder ACLs.
- Sync Entra ID users/groups.
- Maintain external identity mapping.
- Support delta queries/change notifications for permission changes.
- Store external ACL hash/version on every source object.

For SaaS systems:

- Prefer native access-control APIs when available.
- If the SaaS has role/team/project permissions, map them to canonical groups/relationships.
- If permissions cannot be reliably represented, mark source as restricted and require source-side query delegation or admin-only indexing.

## Permission propagation

A document produces multiple derived artifacts:

- chunks
- embeddings
- summaries
- OCR text
- image captions
- table exports
- graph nodes/edges
- cached answer snippets

Every derived artifact must inherit the source document ACL unless explicitly downgraded to a stricter policy.

Rule: derived artifacts can be equal or more restrictive than the source, never less restrictive.

## Retrieval-time enforcement

The retrieval pipeline must filter before model context construction:

```text
request arrives
  ↓
authenticate principal
  ↓
resolve groups/roles/attributes
  ↓
apply tenant/source/document/chunk ACL filters
  ↓
keyword/vector/graph retrieval over authorized candidates only
  ↓
rerank authorized candidates only
  ↓
construct model context from authorized evidence only
  ↓
generate cited answer
  ↓
post-generation guard verifies citations/resources are authorized
```

Vector DB options:

- Store `tenant_id`, `acl_group_ids`, `acl_principal_ids`, `sensitivity`, `source_id` as payload metadata.
- Use payload filters during vector search.
- For large ACL sets, avoid giant per-user filters by indexing permission groups, source scopes, or precomputed access labels.
- Consider per-tenant collections; avoid per-user collections except for small/private stores.

Graph DB:

- Edges and nodes need ACL metadata/provenance.
- Graph traversals must be permission-scoped.
- If an entity is inferred from multiple docs, visibility should depend on whether the requester can see at least one supporting source — and citations must only include visible sources.

## Agent permissions

Agents should be first-class principals, not superusers.

Agent identity:

- Agent has its own `principal_id`.
- Agent may act as itself, or on behalf of a user.
- On-behalf-of mode must include both identities:

```json
{
  "actor": "agent:agentcrm",
  "subject": "user:nick",
  "mode": "on_behalf_of",
  "scopes": ["search", "read:citation"]
}
```

Authorization should evaluate the intersection of:

- what the human can see
- what the agent is allowed to do
- what the integration/source allows

This prevents an agent from becoming a permission escalation path.

## External federated identity

Federation strategy:

- OIDC/SAML for login.
- SCIM or directory APIs for users/groups sync.
- Microsoft Graph / Google Admin SDK for group and file permission sync.
- Token exchange / on-behalf-of flows where source-side delegation is required.

When possible, preserve source-side permissions exactly. Do not flatten rich source permissions into coarse roles unless unavoidable.

## Answer safety

Generated answers must be evidence-bound:

- Include citations to authorized source chunks.
- If no authorized evidence exists, say so.
- Do not mention hidden document titles or counts if that leaks existence.
- Do not summarize across documents where some documents are unauthorized.
- Cache answers only with the principal/scope/policy hash that produced them.

## Audit logging

Log every security-relevant action:

- who/what asked
- on behalf of whom
- query text
- filters/scopes
- document IDs returned
- citations used
- answer generated
- denied attempts
- policy version

Audit logs should be immutable/append-only and queryable by admins.

## Recommended V1 implementation

V1 should support:

- Supabase Auth or enterprise OIDC.
- Canonical principal/group tables in Postgres.
- Postgres RLS for metadata.
- Qdrant payload filters for vector retrieval.
- Neo4j graph queries scoped by tenant/source/ACL metadata.
- Microsoft Entra ID + Graph connector as the first enterprise directory/source integration.
- Local file ACL ingestion for appliance-native storage.
- Agent API keys/JWTs with scoped permissions.
- Full audit logging.

V1 should avoid:

- Per-user vector indexes.
- Post-generation-only permission filtering.
- Treating agents as global admins.
- Caching answers without principal/scope keys.
- Indexing external SaaS content if source permissions cannot be represented or delegated safely.
