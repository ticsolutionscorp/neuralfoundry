# AI Knowledge NAS — Model Strategy

## Product family

Supports [[neural-vault|Neural Vault]] local inference and [[neural-forge|Neural Forge]] model adaptation. Parent: [[neural-foundry|Neural Foundry]].

## Core conclusion

This product should not depend on a single “best model.” It should use a model stack with separate roles for ingestion/indexing and retrieval/usage. The winning architecture is model-adapter based so the appliance can improve as local models evolve.

Recommended V1 default stack:

- Document parsing/OCR/layout: Docling + Nemotron OCR as primary OCR backend + traditional OCR fallback + Nemotron VL for hard visual pages.
- Embeddings: Nemotron embedding family as the default; Qwen3-Embedding-4B as the quality alternative; BGE-M3 as the efficient/hybrid fallback.
- Reranking: Nemotron reranker family as default; Qwen3-Reranker-4B as alternative; BGE-reranker-v2-m3 for lower-latency installations.
- Knowledge extraction: Nemotron 3 Nano/Super for summaries, entity/relation extraction, labels, and metadata; Qwen3 or GLM 5.1 as alternative profiles.
- Human/agent answering: Nemotron 3 Nano/Super as default local RAG/chat model; Qwen3 MoE as alternative profile; GLM 5.1 profile for agentic workflows and tool-heavy reasoning.
- Optional premium visual reasoning: Nemotron VL or Qwen3-VL 32B where hardware supports it.

## Phase 1 — crawling, understanding, indexing

The ingestion/indexing phase needs multiple model types:

1. Parsers and OCR
   - Docling should be the primary document intelligence layer.
   - Nemotron OCR should be the default OCR backend via Docling integration.
   - Use standard OCR for straightforward scanned documents.
   - Use VLM only where layout, tables, charts, handwriting, screenshots, or diagrams need semantic interpretation.

2. Vision-language model
   - Default: Nemotron VL.
   - Alternative: Qwen3-VL.
   - Role: understand screenshots, charts, dense PDFs, tables, figures, diagrams, and pages where raw OCR loses meaning.
   - Practical default: Nemotron VL or Qwen3-VL 8B/32B depending on appliance tier.

3. Embedding model
   - Recommended default: Nemotron embedding family.
   - Quality alternative: Qwen3-Embedding-4B.
   - Premium/maximum quality: Qwen3-Embedding-8B.
   - Efficient fallback: BGE-M3.
   - Role: produce vectors for semantic retrieval. Store model name/version/dimensions with every embedding so reindexing is manageable.

4. Reranker
   - Recommended default: Nemotron reranker family.
   - Quality alternative: Qwen3-Reranker-4B.
   - Efficient fallback: BGE-reranker-v2-m3.
   - Role: improve top-k retrieval quality before sending context to the final answer model.

5. Metadata/entity extraction model
   - Recommended default: Nemotron 3 Nano/Super.
   - Alternative: Qwen3 local LLM or GLM 5.1.
   - Role: produce document summaries, tags, topics, entities, relationships, dates, organizations, people, products, projects, sensitivity labels, and graph edges.
   - Important: extraction should be structured JSON with confidence scores and source spans, not freeform summaries only.

## Phase 2 — ongoing usage by humans and AI agents

The usage phase needs models optimized for grounded answers, tool use, citation discipline, and latency.

1. Retrieval model stack
   - Hybrid query: keyword + vector + graph + permissions.
   - Rerank retrieved chunks.
   - Construct answer context with citations, source spans, and permission checks.

2. Answer model
   - Recommended default: Nemotron 3 Nano/Super.
   - Reason: strong NVIDIA-optimized local RAG performance, integrated with the Nemotron OCR/embedding/reranking stack.
   - Alternative profile: Qwen3 MoE for non-NVIDIA hardware or where MoE efficiency matters.
   - Agentic profile: Nemotron 3 Super or GLM 5.1 where tool use, planning, and multi-step reasoning matter most.

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

### Efficient appliance (non-NVIDIA / low-VRAM)

- OCR: Docling + Tesseract/PaddleOCR.
- Embeddings: BGE-M3 or Qwen3-Embedding-0.6B/4B.
- Reranker: BGE-reranker-v2-m3.
- VLM: Qwen3-VL 8B only for selected hard pages.
- Answer model: smaller Qwen3 instruct/MoE variant.

### Recommended appliance (NVIDIA GPU)

- OCR: Docling + Nemotron OCR.
- Embeddings: Nemotron embedding family.
- Reranker: Nemotron reranker family.
- VLM: Nemotron VL / Qwen3-VL 8B.
- Extraction/chat: Nemotron 3 Nano/Super + optional GLM 5.1 agent profile.

### Premium RTX PRO 6000 appliance

- OCR: Docling + Nemotron OCR.
- Embeddings: Nemotron embedding family for maximum quality.
- Reranker: Nemotron reranker family.
- VLM: Nemotron VL / Qwen3-VL 32B for visual/doc understanding, with smaller VLM workers for bulk jobs.
- Answer/agent model: Nemotron 3 Super default; GLM 5.1 agentic mode; larger models can be served via vLLM/SGLang/TensorRT-LLM/NVIDIA Dynamo.

## Product principle

Model selection must be configurable per deployment. Store model lineage in metadata and make reindexing/re-embedding a first-class maintenance operation. The appliance should ship with sane defaults (Nemotron family) but never hard-code the knowledge base to one model family.

## Alternative stack — Qwen3/BGE profile (non-NVIDIA hardware)

When NVIDIA hardware or Nemotron models are not available, the appliance defaults to a Qwen3/BGE profile:

- Document framework: Docling.
- OCR: Tesseract / PaddleOCR (Docling OCR backend).
- Embeddings: Qwen3-Embedding-4B quality default; BGE-M3 efficient fallback.
- Reranking: Qwen3-Reranker-4B quality default; BGE-reranker-v2-m3 efficient fallback.
- Visual/document reasoning: Qwen3-VL 8B/32B.
- Answer model: Qwen3 MoE.
- Agentic mode: GLM 5.1.
- Serving/runtime: Ollama, vLLM, SGLang.

Sources:
- Docling at NVIDIA GTC / Nemotron OCR integration: https://www.docling.ai/blog/20260311_00_docling_at_gtc/?filter=all
- NVIDIA NeMo Retriever / Nemotron RAG ecosystem: https://developer.nvidia.com/nemo-retriever
- NVIDIA Nemotron foundation models: https://www.nvidia.com/en-us/ai-data-science/foundation-models/nemotron/
