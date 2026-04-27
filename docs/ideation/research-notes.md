# AI Knowledge NAS — Research Notes

## Product family

Research supporting [[neural-foundry|Neural Foundry]] and related modules: [[neural-vault|Neural Vault]], [[neural-forge|Neural Forge]], [[neural-lens|Neural Lens]].

## Initial Landscape Notes

Research found a strong open-source ecosystem to compose rather than reinvent.

### Ingestion / Parsing

- Apache Tika: mature content detection/extraction across many file formats; useful broad fallback.
- Unstructured: RAG-oriented document partitioning and connectors; mature but may be less strong than newer tools for complex layouts.
- Docling: standout candidate for structured document intelligence, layout, tables, OCR, multimodal RAG preparation, and integrations with LangChain/LlamaIndex/Haystack.
- Docling-Graph / Semantica: promising paths for converting parsed documents into graph structures.

### GraphRAG / Retrieval

- Neo4j + Qdrant is a common production-friendly hybrid: graph for relationships/traversal, vector DB for semantic recall.
- ArangoDB is a possible simplified multi-model alternative combining document/graph/vector in one system.
- LlamaIndex appears especially relevant for property graph indexing and document-centric retrieval.
- LangChain/LangGraph remains useful for agent workflows and graph extraction patterns.
- Haystack is worth evaluating for production RAG pipeline observability.

## Sources from initial search

- Apache Tika: https://tika.apache.org/
- Unstructured: https://unstructured.io/
- Docling: https://github.com/docling-project/docling
- Docling-Graph: https://github.com/docling-project/docling-graph
- RAGFlow: https://github.com/infiniflow/ragflow
- Qdrant + Neo4j GraphRAG: https://qdrant.tech/documentation/examples/graphrag-qdrant-neo4j/
- Neo4j + LlamaIndex PropertyGraphIndex: https://neo4j.com/blog/developer/property-graph-index-llamaindex/
- Neo4j + LangChain GraphRAG: https://neo4j.com/blog/developer/global-graphrag-neo4j-langchain/

## Early Research Conclusion

The likely winning approach is not to build a bespoke RAG stack from scratch. Build a unified product layer around proven components: Docling/Tika/Unstructured for ingestion; PostgreSQL/object storage for durable metadata/files; Qdrant/Neo4j or ArangoDB for retrieval/knowledge; local inference workers for extraction; and clean human/agent interfaces above it.

## MinIO viability check — 2026-04-25

MinIO should be treated carefully for new AI Knowledge NAS deployments. The original `minio/minio` GitHub repository is now archived/read-only as of Apr 25, 2026, while the project/company is steering users toward commercial AIStor offerings. The code is AGPLv3, which may be acceptable for internal/self-hosted usage but creates compliance concerns for commercial distribution or hosted SaaS.

For V1 architecture, prefer an S3-compatible storage abstraction rather than hard-coding MinIO. Candidate backends: SeaweedFS, Garage, RustFS, Ceph RGW, cloud S3/Azure Blob, or a maintained MinIO fork such as pgsty/minio/Silo if legal/compliance accepts AGPL.

Sources:
- Original MinIO repo archived/read-only: https://github.com/minio/minio
- MinIO AIStor pricing/product direction: https://www.min.io/pricing
- MinIO AGPL license announcement: https://www.min.io/blog/from-open-source-to-free-and-open-source-minio-is-now-fully-licensed-under-gnu-agplv3
- Community fork: https://github.com/pgsty/minio
- SeaweedFS alternative: https://github.com/seaweedfs/seaweedfs
