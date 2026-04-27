# AI Knowledge NAS — Model Strategy

## Product family

Supports [[neural-vault|Neural Vault]] local inference and [[neural-forge|Neural Forge]] model adaptation. Parent: [[neural-foundry|Neural Foundry]].

## Core conclusion

This product should not depend on a single “best model.” It should use a model stack with separate roles for ingestion/indexing and retrieval/usage. The winning architecture is model-adapter based so the appliance can improve as local models evolve.

Recommended V1 default stack:

- Document parsing/OCR/layout: Docling + traditional OCR fallback + Qwen3-VL for hard visual pages.
- Embeddings: Qwen3-Embedding-4B as the quality default; BGE-M3 as the efficient/hybrid fallback.
- Reranking: Qwen3-Reranker-4B for quality; BGE-reranker-v2-m3 for lower-latency installations.
- Knowledge extraction: Qwen3 or GLM 5.1 class local LLM for summaries, entity/relation extraction, labels, and metadata.
- Human/agent answering: Qwen3 MoE as default local RAG/chat model; GLM 5.1 profile for agentic workflows and tool-heavy reasoning.
- Optional premium visual reasoning: Qwen3-VL 32B or larger where hardware supports it.

## Phase 1 — crawling, understanding, indexing

The ingestion/indexing phase needs multiple model types:

1. Parsers and OCR
   - Docling should be the primary document intelligence layer.
   - Use standard OCR for straightforward scanned documents.
   - Use VLM only where layout, tables, charts, handwriting, screenshots, or diagrams need semantic interpretation.

2. Vision-language model
   - Recommended: Qwen3-VL.
   - Role: understand screenshots, charts, dense PDFs, tables, figures, diagrams, and pages where raw OCR loses meaning.
   - Practical default: Qwen3-VL 8B or 32B depending on appliance tier.

3. Embedding model
   - Recommended quality default: Qwen3-Embedding-4B.
   - Premium/maximum quality: Qwen3-Embedding-8B.
   - Efficient fallback: BGE-M3.
   - Role: produce vectors for semantic retrieval. Store model name/version/dimensions with every embedding so reindexing is manageable.

4. Reranker
   - Recommended quality default: Qwen3-Reranker-4B.
   - Efficient fallback: BGE-reranker-v2-m3.
   - Role: improve top-k retrieval quality before sending context to the final answer model.

5. Metadata/entity extraction model
   - Recommended: Qwen3 local LLM or GLM 5.1.
   - Role: produce document summaries, tags, topics, entities, relationships, dates, organizations, people, products, projects, sensitivity labels, and graph edges.
   - Important: extraction should be structured JSON with confidence scores and source spans, not freeform summaries only.

## Phase 2 — ongoing usage by humans and AI agents

The usage phase needs models optimized for grounded answers, tool use, citation discipline, and latency.

1. Retrieval model stack
   - Hybrid query: keyword + vector + graph + permissions.
   - Rerank retrieved chunks.
   - Construct answer context with citations, source spans, and permission checks.

2. Answer model
   - Recommended default: Qwen3 MoE/local instruct model.
   - Reason: strong local RAG performance, long context, good multilingual handling, practical local deployment.
   - Alternative profile: GLM 5.1 for agentic workflows where tool use, planning, and multi-step reasoning matter more than generic chat.

3. Agent-facing mode
   - Agents should not just chat with the NAS. They should call API tools:
     - search(query, filters)
     - get_document(id)
     - get_chunk(id)
     - explain_citation(id)
     - traverse_graph(entity)
     - watch_collection(path/project)
   - The model should generate plans and interpret results, but retrieval should remain deterministic and auditable.

4. Human-facing mode
   - Prioritize concise answers, citations, source previews, and confidence.
   - Default response should say when evidence is weak or missing.
   - The UI should expose “why this result?” and “open source file.”

## Appliance model profiles

### Efficient appliance

- Embeddings: BGE-M3 or Qwen3-Embedding-0.6B/4B.
- Reranker: BGE-reranker-v2-m3.
- VLM: Qwen3-VL 8B only for selected hard pages.
- Answer model: smaller Qwen3 instruct/MoE variant.

### Recommended appliance

- Embeddings: Qwen3-Embedding-4B.
- Reranker: Qwen3-Reranker-4B.
- VLM: Qwen3-VL 8B/32B.
- Extraction/chat: Qwen3 MoE + optional GLM 5.1 agent profile.

### Premium RTX PRO 6000 appliance

- Embeddings: Qwen3-Embedding-8B for maximum quality, or 4B if throughput matters more.
- Reranker: Qwen3-Reranker-4B/8B.
- VLM: Qwen3-VL 32B for visual/doc understanding, with smaller VLM workers for bulk jobs.
- Answer/agent model: Qwen3 MoE default; GLM 5.1 agentic mode; larger models can be served via vLLM/SGLang/TensorRT-LLM.

## Product principle

Model selection must be configurable per deployment. Store model lineage in metadata and make reindexing/re-embedding a first-class maintenance operation. The appliance should ship with sane defaults but never hard-code the knowledge base to one model family.

## Alternative stack — standardized NVIDIA Nemotron profile

Nick asked what the stack would look like if we standardized on Nemotron models after reviewing Docling's NVIDIA GTC post.

A Nemotron-standardized appliance is a very coherent option, especially if the hardware target is NVIDIA RTX PRO / Blackwell and we want strong vendor-optimized local inference.

Recommended Nemotron profile:

- Document framework: Docling.
- OCR: NVIDIA Nemotron OCR via Docling OCR backend.
- Multimodal document/page embeddings: NVIDIA Nemotron / NeMo Retriever VL embedding models where available.
- Text embeddings: NVIDIA Nemotron embedding model family, with Qwen3/BGE retained only as optional non-NVIDIA adapters.
- Reranking: NVIDIA Nemotron text/VL reranker family.
- Visual/document reasoning: Nemotron Nano VL / Nemotron VL model family for visually complex pages, screenshots, charts, and diagrams.
- Answer model: Nemotron 3 Nano/Super depending on appliance tier.
- Agentic mode: Nemotron 3 Super or larger profile when hardware supports it.
- Serving/runtime: NVIDIA NIM, TensorRT-LLM, Dynamo, Triton, or vLLM/SGLang where support is strongest.

Why this is attractive:

- Strong hardware/software alignment for RTX PRO 6000 Blackwell-class appliance.
- Unified NVIDIA support story across OCR, embedding, reranking, VLM, and LLM serving.
- Nemotron OCR integrated with Docling showed roughly 60% average throughput improvement in Docling's GTC post while maintaining or improving retrieval quality on ViDoRe v3.
- Better product positioning for an enterprise appliance: NVIDIA-accelerated document AI stack rather than a grab bag of unrelated open models.

Risks / caveats:

- More NVIDIA ecosystem dependency.
- Need to verify licenses/model terms for each exact Nemotron component.
- Need empirical benchmarks against Qwen3-Embedding/Reranker and Qwen3-VL on our target corpus.
- Availability and local deployment paths may vary by model; some components may be easiest via NIM/NGC rather than simple Hugging Face/local runners.

Practical recommendation: make "Nemotron Appliance Profile" the premium/default NVIDIA hardware profile, but keep the generic model adapter interface. Do not remove Qwen/BGE adapters; use them as benchmark comparators and non-NVIDIA fallback profiles.

Sources:
- Docling at NVIDIA GTC / Nemotron OCR integration: https://www.docling.ai/blog/20260311_00_docling_at_gtc/?filter=all
- NVIDIA NeMo Retriever / Nemotron RAG ecosystem: https://developer.nvidia.com/nemo-retriever
- NVIDIA Nemotron foundation models: https://www.nvidia.com/en-us/ai-data-science/foundation-models/nemotron/
