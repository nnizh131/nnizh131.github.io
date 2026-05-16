---
title: RAG Pipeline
description: A retrieval-augmented generation pipeline for question answering over private document collections.
github: https://github.com/nikanizharadze/rag-pipeline
draft: false
tags: [Python, LLM, RAG]
---

## Overview

Large language models hallucinate when asked about information outside their training data. RAG fixes this by retrieving relevant context from a document store before generating an answer, grounding the model's output in real sources.

## Architecture

```
Query → Embed → Retrieve (FAISS) → Rerank → Prompt → LLM → Answer
```

The pipeline supports PDF, markdown, and plain text ingestion. Documents are chunked with overlap, embedded, and stored. At query time, top-k chunks are retrieved and injected into the prompt as context.

## Key design decisions

- **Chunk overlap** — 20% overlap between chunks prevents answers from being cut across boundaries.
- **Hybrid retrieval** — dense + sparse (BM25) scores are fused with Reciprocal Rank Fusion for better recall.
- **Source citations** — the prompt instructs the model to cite chunk IDs; these are resolved to document names and page numbers in post-processing.

## Stack

- Python, LangChain, FAISS
- Anthropic Claude for generation
- Streamlit for a minimal demo UI
