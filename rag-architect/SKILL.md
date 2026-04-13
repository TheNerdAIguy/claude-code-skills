---
name: rag-architect
description: Use when building anything with vector search, embeddings, or a knowledge base — designs the complete RAG pipeline including chunking, embeddings, retrieval, and reranking
---

# RAG Architect

## Overview

Building a RAG (Retrieval-Augmented Generation) system without a clear architecture leads to poor retrieval quality, high latency, and expensive failures at scale. This skill designs the complete pipeline before you write a line of code.

**Core principle:** Retrieval quality is the #1 determinant of RAG output quality. Design it deliberately, not by default.

**Announce at start:** "I'm using the rag-architect skill to design the pipeline before implementation."

## When to Use

Use when building:
- Document Q&A systems
- Knowledge base chatbots
- Semantic search engines
- Code search / navigation
- Product recommendation (semantic)
- Any LLM app that needs to retrieve context from a corpus

## Phase 1 — Define Requirements

Answer these before designing:

1. **What is the corpus?** (PDFs, web pages, code files, database records, mixed?)
2. **How large is the corpus?** (KB → MB → GB → TB)
3. **How often does content change?** (Static, daily, real-time?)
4. **What are the queries like?** (Short keywords, long questions, multi-hop reasoning?)
5. **What language(s)?** (Mono-lingual, multi-lingual?)
6. **What's acceptable latency?** (< 500ms? < 2s?)
7. **What's the budget?** (Embedding costs, vector DB costs, LLM calls?)
8. **What's the accuracy requirement?** (Best-effort vs. mission-critical)

## Phase 2 — Pipeline Design

### 2.1 Document Ingestion

```
Raw Document → Loader → Cleaner → Chunker → Embedder → Vector Store
```

**Loader** — Choose by source:
| Source | Loader |
|--------|--------|
| PDF | PyPDF2, pdfplumber (for complex layouts), Azure Doc Intelligence |
| HTML/Web | BeautifulSoup, Firecrawl, markitdown |
| DOCX/PPTX | python-docx, python-pptx, markitdown |
| Code | Tree-sitter (structure-aware), plain text |
| Database | Direct SQL + template formatting |

**Cleaner** — Always do:
- Remove headers/footers/page numbers
- Normalize whitespace
- Remove duplicate sections
- Handle encoding issues

### 2.2 Chunking Strategy

**Choose based on content type:**

| Content | Strategy | Chunk Size | Overlap |
|---------|----------|------------|---------|
| Narrative text | Recursive character | 512-1024 tokens | 10-20% |
| Technical docs | Semantic (by heading) | Variable | Minimal |
| Code | Function/class boundaries | Variable | None |
| Tables | Row or full table | Full | None |
| Q&A pairs | One pair per chunk | Variable | None |

**Chunking rules:**
- Smaller chunks = better precision, more retrieval calls
- Larger chunks = more context per chunk, harder to rank relevance
- Always preserve metadata: source, page, section, date
- Use parent-child chunking for long documents (retrieve child, return parent)

### 2.3 Embedding Model Selection

| Use Case | Model | Dimensions | Notes |
|----------|-------|------------|-------|
| General English | `text-embedding-3-small` (OpenAI) | 1536 | Good default |
| High accuracy | `text-embedding-3-large` (OpenAI) | 3072 | 2x cost |
| Multi-lingual | `multilingual-e5-large` | 1024 | Open source |
| Code | `voyage-code-2` | 1536 | Specialized |
| Low latency / free | `all-MiniLM-L6-v2` | 384 | Local, fast |

### 2.4 Vector Database Selection

| Scale | Options | Notes |
|-------|---------|-------|
| < 1M vectors (prototype) | pgvector (Supabase/Postgres) | No extra infra |
| 1M–50M vectors | Qdrant, Weaviate, Chroma | Self-hosted or managed |
| 50M+ vectors | Pinecone, Weaviate Cloud | Managed, enterprise |
| Serverless | Pinecone serverless, Turbopuffer | Pay-per-query |

### 2.5 Retrieval Strategy

**Baseline: Dense Retrieval**
```python
# Query → embed → ANN search → top-k chunks
results = vectordb.query(query_embedding, top_k=20)
```

**Better: Hybrid Retrieval (Dense + Sparse)**
```python
# Combine semantic search with keyword search
dense_results = vectordb.dense_search(query_embedding, top_k=20)
sparse_results = vectordb.bm25_search(query_text, top_k=20)
results = reciprocal_rank_fusion(dense_results, sparse_results)
```

**Advanced: Multi-Query Retrieval**
```python
# LLM generates N variations of the query, retrieve for each
queries = llm.generate_variations(original_query, n=3)
all_results = [vectordb.search(q) for q in queries]
results = deduplicate_and_merge(all_results)
```

### 2.6 Reranking (Critical for Quality)

Always rerank before passing to LLM:

```python
from cohere import Client
co = Client(api_key)

# Rerank top-20 down to top-5
reranked = co.rerank(
    query=user_query,
    documents=[r.text for r in results],
    top_n=5,
    model='rerank-english-v3.0'
)
```

**Reranking options:**
- Cohere Rerank (best managed)
- BGE Reranker (open source, self-host)
- Cross-encoder models (local, accurate)

### 2.7 Context Assembly

```python
# Assemble context for LLM
context = "\n\n---\n\n".join([
    f"Source: {r.metadata['source']}\n{r.text}"
    for r in top_results
])

prompt = f"""Answer based on the following context:

{context}

Question: {user_query}
Answer:"""
```

**Rules:**
- Include source attribution in context
- Order by relevance (reranked score)
- Respect token budget (leave room for response)
- Handle "no relevant context found" gracefully

## Phase 3 — Evaluation

Before going live, evaluate with a test set:

| Metric | Tool | Target |
|--------|------|--------|
| Retrieval recall@k | RAGAS | > 0.8 |
| Context relevance | RAGAS | > 0.7 |
| Answer faithfulness | RAGAS | > 0.85 |
| Latency P95 | Custom | < 2s |

```python
from ragas import evaluate
from ragas.metrics import context_recall, faithfulness, answer_relevancy

results = evaluate(dataset, metrics=[context_recall, faithfulness, answer_relevancy])
```

## Complete Architecture Diagram

```
User Query
    ↓
[Query Processing]
    ├── Query rewriting / HyDE
    └── Multi-query expansion
         ↓
[Vector Search] + [BM25 Search]
    ↓
[Reciprocal Rank Fusion]
    ↓
[Cross-Encoder Reranking]
    ↓
[Context Assembly + Source Attribution]
    ↓
[LLM Generation]
    ↓
[Response + Citations]
```

## Activation Phrases

Claude activates this skill when you say:
- "design a RAG system" / "vector search" / "embeddings pipeline"
- "build a knowledge base chatbot"
- "semantic search over documents"
- "AI Q&A over my documents/codebase"
