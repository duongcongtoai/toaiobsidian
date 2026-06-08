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
This note defines Out-of-Order Execution (OoO), a CPU optimization technique that mitigates stalls by reordering instructions to utilize idle pipeline slots while waiting for memory access.

# Out-of-Order Execution (OoO)

Out-of-Order Execution is a CPU optimization designed to improve [[CPU Pipeline Slots]] utilization.

Instead of waiting for a slow instruction to complete (like a LOAD that misses the cache and must fetch from DRAM), the CPU looks ahead in the instruction stream. It finds independent instructions that do not rely on the stalled data and executes them immediately.

For example, if a `LOAD A` causes a cache miss, but the subsequent `ADD B` and `MUL C` do not depend on `A`, the CPU will execute the addition and multiplication while waiting for `A` to arrive.

This prevents the CPU from wasting valuable cycles sitting idle, thereby reducing Backend Bound stalls as identified in [[Top-Down Microarchitecture Analysis]].