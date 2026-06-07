---
type: concept
date: 2026-06-07
tags:
  - concept
  - performance
  - hardware
  - cpu
sources:
  - "[[raw/articles/2026-06-07 — Rust Performance Understanding Cache Behavior Beyond Benchmarks.md]]"
ai-first: true
---
## For future Claude
This note outlines the distinction between software simulation and hardware measurement for analyzing CPU cache behavior, particularly why simulators fail to model features like hardware prefetchers, and why PMU-based measurements are preferred for real-world optimization.

# CPU Cache: Simulation vs Measurement

When analyzing memory access patterns and CPU cache performance, there are two primary approaches:

## Simulation

Software simulators model cache hierarchy behavior deterministically.

*   **Tools:** iai-callgrind, Valgrind Cachegrind, gem5, Sniper.
*   **Pros:** Reproducible, deterministic, no measurement noise. Great for A/B testing two implementations.
*   **Cons:** Cannot accurately model real-world, complex CPU features like **[[Hardware Prefetcher]]s**, speculative execution, NUMA effects, or cache replacement policies.

Simulators answer: *"What would happen under this cache model?"* but not *"What actually happened on my CPU?"*

## Hardware Measurement

Hardware measurement uses the CPU's Performance Monitoring Unit (PMU) to collect actual hardware events during execution.

*   **Tools:** perf (e.g., `perf stat`, `perf record`), Intel VTune, AMD uProf.
*   **Metrics:** L1/L2/L3 cache misses, branch mispredictions, IPC (Instructions Per Cycle), etc.

Because real hardware measurements account for actual [[Hardware Prefetcher]] behavior and actual memory layouts, they are vital for diagnosing real-world [[Cache Pollution]] or observing the tangible benefits of contiguous memory access (e.g., as seen in the [[Implement tight loop benchmark]] and [[Prove L2-sized hashtables outperform large ones]] exercises).