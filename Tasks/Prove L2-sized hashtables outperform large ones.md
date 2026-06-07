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
A take-home exercise to prove that L2-sized hash tables outperform large ones due to cache locality and demand fetching.

# Prove L2-sized hashtables outperform large ones
*Extracted from: [[raw/articles/2026-06-07 — Hardware-Level Performance and DataFusion Aggregation.md]]*

Write a benchmark script that allocates a 100KB hash table and a 1GB hash table, does 10 million random lookups in both, and compares the throughput/latency to directly observe cache misses in action. This demonstrates why [[DataFusion]] adaptive skipping is necessary.