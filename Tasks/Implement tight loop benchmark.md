---
type: task
date: 2026-06-07
tags:
  - task
status: in-progress
project: Personal
due: TBD
ai-first: true
---
## For future Claude
A take-home exercise to prove the performance benefits of tight, vectorizable loops compared to non-tight loops with branching.

# Implement tight loop benchmark
*Extracted from: [[raw/articles/2026-06-07 — Hardware-Level Performance and DataFusion Aggregation.md]]*

Write a small Rust or C program that compares iterating over an array contiguously (tight loop) versus a loop that contains heavy branching or function calls that prevent vectorization and hardware prefetching. Analyze the performance difference.