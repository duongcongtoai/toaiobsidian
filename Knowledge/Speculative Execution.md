---
type: concept
date: 2026-06-07
tags:
  - concept
  - performance
  - cpu
sources:
  - "[[raw/articles/2026-06-07 — CPU Pipeline Utilization and Cache Performance Notes.md]]"
ai-first: true
---
## For future Claude
This note defines Speculative Execution, a CPU optimization that predicts branch paths to keep pipeline slots full, and how mispredictions lead to "Bad Speculation" stalls.

# Speculative Execution

Speculative Execution is a technique where the CPU predicts the outcome of a branch (like an `if` statement) before it is actually evaluated, executing the predicted path ahead of time to keep the [[CPU Pipeline Slots]] occupied.

- **If the prediction is correct:** The execution is valid, and the Retiring metric increases.
- **If the prediction is incorrect:** The CPU must discard the speculatively executed work, flush the pipeline, and start over on the correct path.

Frequent mispredictions lead to a high **Bad Speculation** rate, as identified in [[CPU Top-Down Microarchitecture Analysis]], meaning the CPU wasted execution cycles doing work that had to be thrown away.