---
type: concept
date: 2026-06-07
tags:
  - concept
  - performance
  - profiling
  - tools
sources:
  - "[[raw/articles/2026-06-07 — Rust Performance Understanding Cache Behavior Beyond Benchmarks.md]]"
ai-first: true
---
## For future Claude
This note categorizes and compares various performance profiling tools used in systems programming and Rust, detailing their intended purposes and whether they measure real hardware behavior or simulate it.

# Performance Profiling Tools

Profiling tools generally fall into distinct categories based on what they measure (time spent vs. hardware stalls) and how they measure it (real PMU vs. simulation).

## Sampling Profilers (Hotspots)
These tools answer *"Where is the program spending time?"*
*   **Samply:** Excellent for flame graphs, thread analysis, and understanding CPU time distribution on macOS/Linux.
*   **cargo flamegraph:** Standard tool for finding hot functions in Rust.

## Hardware Measurement (PMU)
These tools answer *"Why is the CPU stalled?"* by measuring real hardware events (cache misses, branch mispredictions) using the Performance Monitoring Unit. Essential for analyzing [[Cache Pollution]] and [[Hardware Prefetcher]] impact.
*   **perf stat:** Hardware counters overview (IPC, overall cache misses).
*   **perf record:** Hotspot analysis augmented with PMU data.
*   **Intel VTune / AMD uProf:** Deep, vendor-specific microarchitectural analysis that can pinpoint L1/L2/L3 or DRAM bounds directly to source code lines.

## Simulation
These tools answer *"What would happen under a specific cache model?"* They are deterministic but fail to model complex real-world hardware behaviors (see [[CPU Cache - Simulation vs Measurement]]).
*   **iai-callgrind / Valgrind Cachegrind:** Simulated cache analysis.
*   **gem5 / Sniper:** Full system/CPU simulators.

## Assembly Inspection
*   **cargo asm:** Used to inspect the actual LLVM-generated assembly, verifying if loops were unrolled, vectorized, or evaluating the number of memory loads executed.