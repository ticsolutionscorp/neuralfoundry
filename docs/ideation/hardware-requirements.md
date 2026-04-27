# AI Knowledge NAS — Hardware Requirements

## Product family

Supports [[neural-vault|Neural Vault]], part of [[neural-foundry|Neural Foundry]].

**Date:** 2026-04-25
**Scope:** Appliance hardware sizing for self-hosted AI Knowledge NAS, with emphasis on GPU/AI accelerator requirements.

## Executive Summary

The AI Knowledge NAS can run without a GPU for basic text indexing and search, but a GPU becomes important if the appliance is expected to feel responsive while continuously parsing PDFs, OCRing scans, transcribing audio/video, captioning images/keyframes, extracting entities/relations, and serving local chat/RAG. The recommended appliance design should support multiple hardware tiers rather than one fixed spec.

**Recommended practical V1 appliance:** 16–32 CPU cores, 128–256 GB RAM, NVMe scratch, redundant bulk storage, and one NVIDIA GPU with at least 24–32 GB VRAM. DGX Spark is attractive for large local models and multimodal/privacy-first deployments because of its 128 GB unified memory, but a workstation/server with RTX 5090 or L40S may be faster for smaller models and high-throughput indexing.

## Workload Categories

1. **Storage and metadata**
   - Object/file storage backend.
   - Supabase/Postgres metadata.
   - Vector database.
   - Graph database.
   - Audit/indexing state.

2. **Ingestion and parsing**
   - File watching and connector crawling.
   - Docling/Tika/Unstructured parsing.
   - OCR for scans/images.
   - Audio transcription.
   - Video keyframe extraction and transcription.

3. **AI enrichment**
   - Embeddings.
   - Classification/tagging.
   - Summaries.
   - Entity/relation extraction.
   - Image and video captioning.
   - Optional local chat/RAG generation.

4. **Serving**
   - Human UI search/chat.
   - Agent CLI/API calls.
   - Permission-aware retrieval.
   - Background indexing jobs.

## Hardware Tiers

### Tier -1 — Personal Neural Vault Lite

Use for one personal OpenClaw agent or at most two light agents, mostly notes, docs, email exports, Markdown, PDFs, and light images/audio. This is the smallest GPU-backed install that still keeps the core local.

Recommended minimum:

- CPU: 8–12 modern cores.
- RAM: 64 GB preferred; 32 GB is possible only with aggressive limits and lighter components.
- GPU: NVIDIA 12 GB VRAM minimum; 16 GB preferred.
  - Budget examples: RTX 3060 12 GB, RTX 4060 Ti 16 GB, RTX A4000 16 GB, used RTX 3080 12 GB.
- Storage:
  - 1–2 TB NVMe for OS, database, vector/graph indexes, cache, and active corpus.
  - 4–12 TB SSD/HDD bulk storage if acting as a true local NAS.
- Network: 1 GbE acceptable for personal; 2.5 GbE nice; 10 GbE only if serving large files/media.

Recommended software profile:

- Object/file storage: local filesystem or single-node S3-compatible backend.
- Metadata: PostgreSQL.
- Vector: pgvector or Qdrant depending on corpus size and retrieval needs.
- Graph: lightweight Neo4j configuration or Kùzu/embedded graph profile for very small installs; full Neo4j when RAM allows.
- LLM serving: Ollama or vLLM/SGLang with one small quantized local model.
- Models: 7B–8B quantized chat/RAG model, lightweight embeddings, lightweight reranker, faster-whisper small/medium or batched transcription.

Capabilities:

- Personal/small-business agent memory and context engine.
- Local semantic search and context pack building.
- Light entity extraction and entity resolution.
- Basic ontology/rule storage.
- Small local LLM for summarization, extraction, and simple RAG.
- Background indexing with constrained concurrency.

Limitations:

- Not ideal for continuous video indexing or heavy OCR at scale.
- 12 GB VRAM limits local model size and context length.
- 32 GB RAM forces compromises: avoid running every component heavily at once, keep indexing concurrency low, and limit model size/context.
- The product should still use the full Neural Vault software stack across tiers for build and maintenance simplicity; avoid a separate Lite architecture unless performance forces a future profile.
- One or two agents max; not suitable for enterprise multi-agent workloads.

### Tier 0 — CPU-only / lightweight dev

Use for prototypes, demos, or indexing text-heavy corpora slowly.

- CPU: 8 cores minimum; 16 preferred.
- RAM: 32–64 GB.
- GPU: none.
- Storage: NVMe scratch + SSD/HDD bulk.
- AI: CPU embeddings, remote/cloud LLM optional.

Limitations:
- Slow OCR/transcription/multimodal extraction.
- Not suitable for always-on video/audio-heavy indexing.
- Local chat models limited or slow.

### Tier 1 — Small appliance

Good for small businesses, power users, or a few OpenClaw agents sharing Neural Vault as a knowledge base, memory substrate, and context engine.

- CPU: 12–16 cores.
- RAM: 96–128 GB preferred; 64 GB acceptable for light use.
- GPU: NVIDIA 16 GB VRAM minimum; 24 GB preferred.
- Example GPU class: RTX 4060 Ti 16 GB / RTX 4070 Ti Super 16 GB / RTX 4080 16 GB / RTX 4090 24 GB.
- Storage: 2–4 TB NVMe scratch/cache/index tier; 8–40 TB bulk depending on use.

Capabilities:
- Local embeddings comfortably.
- Faster Whisper transcription.
- Docling GPU acceleration for document parsing.
- Small/medium local LLMs and VLMs.

### Tier 2 — Recommended V1 appliance

Best target for serious self-hosted deployments.

- CPU: 16–32 cores.
- RAM: 128–256 GB ECC preferred.
- GPU: one 24–32 GB NVIDIA GPU minimum.
  - RTX 4090 24 GB: strong value.
  - RTX 5090 32 GB: better 2026 sweet spot if available.
  - L40S 48 GB: better server/datacenter card with more VRAM.
- Storage:
  - OS: mirrored SSD.
  - Scratch/cache: 2–8 TB NVMe.
  - Object data: redundant HDD/SSD pool; capacity based on corpus.
  - Optional separate NVMe for Postgres/Qdrant/Neo4j.
- Network: 10 GbE minimum if acting as NAS; 25 GbE nice for multi-user/media workloads.

Capabilities:
- High-throughput document indexing.
- OCR/transcription running continuously.
- Local embeddings/reranking.
- Small/medium local chat/RAG models.
- Multimodal extraction with controlled concurrency.

### Tier 3 — Premium local AI appliance

Use when the appliance should run larger local models, stronger multimodal extraction, or many concurrent agents.

Option A: **DGX Spark profile**
- NVIDIA Grace Blackwell GB10-class system.
- 128 GB unified memory.
- Strong fit for large models, privacy-first local inference, and OSFoundry-style deployments.
- Better for fitting large models than for raw decode speed versus high-bandwidth discrete GPUs.

Option B: **Server GPU profile**
- 1–2x L40S 48 GB or equivalent datacenter GPUs.
- 256–512 GB system RAM.
- 8–16 TB NVMe scratch/cache.
- 10/25/40 GbE.

Capabilities:
- Larger local LLM/VLM models.
- More concurrent ingestion and chat workloads.
- Heavier video/image understanding.
- Better client/on-prem enterprise profile.

### Tier 4 — Cluster / enterprise

Use for tens/hundreds of TB or many users/agents.

- Separate storage nodes from AI worker nodes.
- Dedicated Postgres/Supabase node.
- Dedicated vector DB and graph DB nodes.
- GPU worker pool with job queue.
- S3-compatible object storage cluster.
- Kubernetes or Nomad optional.

## GPU / Accelerator Guidance

### What the GPU does

The GPU is not primarily for storage. It accelerates:

- Embedding generation.
- OCR/layout/document understanding.
- Audio transcription.
- Image/keyframe captioning.
- Entity/relation extraction using local LLMs.
- Local chat/RAG generation.
- Reranking and classification.

### VRAM rules of thumb

- Embeddings: usually lightweight; many strong embedding models run in under 2 GB VRAM, but throughput benefits from batching.
- faster-whisper large-v3-turbo: roughly 2.5–4 GB VRAM typical; 6 GB comfortable; full large-v3 can need ~10 GB.
- Docling standard parsing/OCR: can run CPU-only; GPU standard pipelines often fit in 1–4 GB VRAM; VLM/advanced pipelines benefit from 8–16+ GB.
- Local 7B–8B LLM/VLM: 8–16 GB depending on quantization/context.
- Local 30B–40B models: 24–32+ GB preferred.
- 70B-class quantized models: 48 GB+ preferred, or unified-memory/offload profiles like DGX Spark.

## Recommended AI Software Stack

- Embeddings: Qwen3-Embedding-0.6B or BGE-M3; evaluate domain quality.
- Transcription: faster-whisper / Whisper large-v3-turbo.
- Document parsing: Docling primary, Tika fallback, optional Unstructured.
- Image/document VLM: small VLM for captioning/extraction; larger model optional on premium tier.
- LLM serving:
  - Ollama for simple appliance mode.
  - vLLM/SGLang/TensorRT-LLM for production inference workers.
  - NVIDIA Dynamo for multi-worker/GPU routing/caching once we have enough local inference traffic.

## Practical Recommendation

For the first real appliance, target:

- CPU: 24–32 cores.
- RAM: 256 GB ECC if possible.
- GPU: RTX 5090 32 GB or L40S 48 GB; RTX 4090 24 GB acceptable for budget/prototype.
- Storage:
  - 2x NVMe mirrored OS.
  - 4–8 TB NVMe scratch/index/cache.
  - Bulk storage sized to corpus, ideally ZFS/RAID with snapshots.
- Network: 10 GbE minimum.

If using existing DGX Spark machines, use them as the **AI worker/inference tier** and keep storage/index services on a NAS/server tier if capacity or disk redundancy requirements exceed the Spark local storage profile.

## Sources

- NVIDIA DGX Spark product page: https://www.nvidia.com/en-us/products/workstations/dgx-spark/
- Docling GPU usage: https://docling-project.github.io/docling/usage/gpu/
- Docling-Graph requirements: https://github.com/docling-project/docling-graph/blob/main/docs/fundamentals/installation/requirements.md
- faster-whisper VRAM discussion: https://huggingface.co/deepdml/faster-whisper-large-v3-turbo-ct2/discussions/3
- OpenAI Whisper performance reference: https://www.mintlify.com/openai/whisper/resources/performance
- BGE-M3 memory discussion: https://huggingface.co/BAAI/bge-m3/discussions/64

## Update — RTX PRO 6000 Blackwell vs DGX Spark

Nick prefers evaluating an NVIDIA RTX PRO 6000 Blackwell card rather than DGX Spark for the AI Knowledge NAS appliance. This is likely the better fit for the primary appliance/inference server if budget, chassis, cooling, and power allow it.

Why:

- RTX PRO 6000 Blackwell provides 96 GB GDDR7 ECC VRAM and far higher memory bandwidth than DGX Spark's 128 GB unified LPDDR5x memory. For LLM inference decode, OCR/VLM throughput, embeddings, reranking, and batch indexing, bandwidth and discrete-GPU throughput matter heavily.
- DGX Spark's advantage is unified memory capacity, compactness, power efficiency, and developer appliance packaging. It is excellent for desktop prototyping or fitting larger models locally, but less ideal as the performance engine of a storage/inference appliance.
- For this project, the RTX PRO 6000 Blackwell profile better matches the desired architecture: storage server + high-throughput AI accelerator + local inference workers.

Revised preferred premium appliance profile:

- CPU: 32+ cores, server/workstation class.
- RAM: 256–512 GB ECC.
- GPU: 1x RTX PRO 6000 Blackwell 96 GB; optionally expandable to 2x if thermals/power/chassis support it.
- Power/cooling: design around ~600 W GPU TDP plus CPU/storage headroom; use workstation/server chassis with strong airflow.
- Storage: NVMe scratch/index tier + redundant bulk storage.
- Network: 10 GbE minimum, 25 GbE preferred.
- Software: vLLM/SGLang/TensorRT-LLM initially; NVIDIA Dynamo if serving multiple model workers or needing KV-aware routing/caching.

Sources:
- NVIDIA RTX PRO desktop GPUs: https://www.nvidia.com/en-us/products/workstations/professional-desktop-gpus/
- NVIDIA DGX Spark: https://www.nvidia.com/en-us/products/workstations/dgx-spark/
