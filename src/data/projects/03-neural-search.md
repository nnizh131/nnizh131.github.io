---
title: Neural Search
description: A semantic search engine that uses dense embeddings to retrieve documents by meaning rather than keywords.
github: https://github.com/nikanizharadze/neural-search
draft: false
tags: [Python, PyTorch, FastAPI]
---

## Overview

Keyword search breaks down the moment a user phrases a query differently from how a document is written. Neural Search replaces BM25 with a bi-encoder retrieval pipeline that maps queries and documents into the same embedding space, so "how do transformers work" and "attention mechanism explained" surface the same results.

## How it works

1. Documents are encoded offline using a fine-tuned sentence transformer and stored in a FAISS index.
2. At query time, the query is encoded with the same model and approximate nearest-neighbour search retrieves the top-k candidates in milliseconds.
3. A cross-encoder reranker scores the shortlist and returns the final ranked results.

```python
from sentence_transformers import SentenceTransformer
import faiss, numpy as np

model = SentenceTransformer("all-MiniLM-L6-v2")

def build_index(docs: list[str]) -> faiss.IndexFlatIP:
    embeddings = model.encode(docs, normalize_embeddings=True)
    index = faiss.IndexFlatIP(embeddings.shape[1])
    index.add(embeddings.astype(np.float32))
    return index
```

## Stack

- Python, PyTorch
- sentence-transformers, FAISS
- FastAPI for the retrieval API
