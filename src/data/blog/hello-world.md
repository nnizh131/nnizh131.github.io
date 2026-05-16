---
title: Hello, World
author: Nika Nizharadze
pubDatetime: 2026-05-14T10:00:00Z
slug: hello-world
featured: true
draft: false
description: First post — why I'm writing and what to expect here.
---

Every blog starts with a "hello world" post, so here's mine.

I've been meaning to have a place to write for a while. Not a newsletter, not a Twitter thread — just a spot where I can think out loud, document things I figure out, and occasionally share something that might be useful to someone else.

## What I'll write about

Mostly engineering. I spend a lot of time thinking about software architecture, developer tooling, and the gap between theory and practice on real projects. Expect posts on those topics, plus whatever rabbit hole I happen to fall down.

I'll also write about non-technical things when the mood strikes. No promises on a schedule.

## Code

Posts will often include code snippets. Here's what that looks like:

```python
import torch
import torch.nn as nn

class Attention(nn.Module):
    def __init__(self, dim: int, heads: int = 8):
        super().__init__()
        self.heads = heads
        self.scale = (dim // heads) ** -0.5
        self.qkv = nn.Linear(dim, dim * 3, bias=False)
        self.proj = nn.Linear(dim, dim)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        B, N, C = x.shape
        qkv = self.qkv(x).reshape(B, N, 3, self.heads, C // self.heads)
        q, k, v = qkv.permute(2, 0, 3, 1, 4).unbind(0)
        attn = (q @ k.transpose(-2, -1)) * self.scale
        attn = attn.softmax(dim=-1)
        return self.proj((attn @ v).transpose(1, 2).reshape(B, N, C))
```

And inline code works too — `torch.nn.functional.scaled_dot_product_attention` is the fused kernel you should be using in practice.

## Math

LaTeX renders inline and as display blocks. The scaled dot-product attention score is:

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

Where $Q$, $K$, $V$ are the query, key, and value matrices and $d_k$ is the key dimension. The $\frac{1}{\sqrt{d_k}}$ factor keeps dot products from growing large in magnitude as dimensionality increases, which would push softmax into regions with tiny gradients.

## Why now

I kept putting this off waiting for the "right" setup. This site is built with [Astro](https://astro.build) and the [AstroPaper](https://github.com/satnaing/astro-paper) theme — simple, fast, and out of my way. Good enough to start.

See you in the next post.
