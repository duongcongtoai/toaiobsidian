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
A take-home exercise to simulate and isolate the performance degradation caused by TLB (Translation Lookaside Buffer) pollution.

# Simulate TLB Pollution
*Extracted from: [[raw/articles/2026-06-07 — Hardware-Level Performance and DataFusion Aggregation.md]]*

Write a C/Rust benchmark that allocates a large array spanning tens of thousands of pages. Compare the performance of accessing elements sequentially (where the [[Translation Lookaside Buffer]] handles 4 KiB chunks smoothly) versus jumping randomly to a new 4 KiB page on every single access (guaranteeing a TLB miss on every read). This will isolate and demonstrate the extreme penalty of TLB thrashing, related to [[Cache Pollution]].