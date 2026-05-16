---
title: LLM Evals Framework
description: A lightweight framework for evaluating language model outputs across custom criteria without relying on human annotation.
github: https://github.com/nikanizharadze/llm-evals
draft: false
tags: [Python, LLM, Evaluation]
---

## Overview

Evaluating LLM outputs at scale is expensive if you rely on humans. This framework uses a judge LLM to score model responses against a rubric, with calibration experiments to estimate agreement with human annotators.

## Key ideas

- **Rubric-based scoring** — each criterion is defined as a prompt template; the judge returns a score and a rationale.
- **Calibration** — a small held-out set of human-labelled examples is used to detect systematic bias in the judge.
- **Async evaluation** — responses are evaluated in parallel to keep costs low.

```python
from evals import Rubric, Judge

rubric = Rubric.from_yaml("rubrics/factuality.yaml")
judge = Judge(model="claude-sonnet-4-6", rubric=rubric)

results = await judge.evaluate_batch(responses)
print(results.summary())
```

## Stack

- Python, asyncio
- Anthropic SDK
- Weights & Biases for tracking
