---
type: concept
date: 2026-06-08
tags:
  - concept
  - performance
  - rust
  - workflow
  - profiling
sources:
  - "[[raw/articles/2026-06-07 — Rust Performance Understanding Cache Behavior Beyond Benchmarks.md]]"
ai-first: true
---
## For future Claude
This note documents a practical, step-by-step performance optimization workflow for Rust applications, tightly integrating the specific profiling tools used at each stage. It transitions from high-level hotspot identification down to deep microarchitectural analysis based on real hardware PMU data rather than simulation.

# Rust Performance Profiling Workflow

A typical real-world Rust performance optimization workflow follows a progressive path from macroscopic analysis to microscopic inspection. Profiling tools generally fall into categories based on what they measure (time spent vs. hardware stalls) and how they measure it (real PMU vs. simulation).

## 1. Find the Hotspot (Macroscopic Analysis)
*   **Goal:** Answer *"Where is the program spending its time?"* Identify the specific function or loop consuming the most CPU time.
*   **Tools:**
    *   **Samply:** Excellent for flame graphs, thread analysis, and understanding CPU time distribution on macOS/Linux.
    *   **cargo flamegraph:** Standard tool for finding hot functions in Rust.

## 2. Measure Real Cache Behavior & Locate Instructions (PMU)
*   **Goal:** Answer *"Why is the CPU stalled?"* by measuring real hardware events (cache misses, branch mispredictions) using the Performance Monitoring Unit (PMU). Combine this with hotspot analysis to see exactly which instructions cause stalls. Essential for analyzing [[Cache Pollution]] and [[Hardware Prefetcher]] impact.
*   **Tools:**
    *   **perf stat -d:** Hardware counters overview (IPC, overall cache misses).
    *   **perf record / perf report:** Hotspot analysis augmented with PMU data.

## 3. Inspect Generated Assembly
*   **Goal:** Answer *"What code is the CPU actually executing?"* Check if LLVM successfully vectorized or unrolled loops, and count memory loads.
*   **Tools:**
    *   **cargo asm:** Used to inspect the actual LLVM-generated assembly for specific crates/functions.

## 4. Deep Microarchitectural Analysis
*   **Goal:** When cache and pipeline behaviors are the primary bottlenecks, attribute stalls (e.g., L1, L2, L3, or DRAM bound) directly to source code lines.
*   **Tools:**
    *   **Intel VTune / AMD uProf:** Deep, vendor-specific microarchitectural analysis tools.

## 5. The Simulation Caveat
While simulation tools exist, they are deterministic but often fail to model complex real-world hardware behaviors (like speculative execution or aggressive prefetching) accurately. See [[CPU Cache - Simulation vs Measurement]].
*   **Tools to use with caution/awareness:** `iai-callgrind`, `Valgrind Cachegrind`, `gem5`, `Sniper`.

----------

## Flashcards

List well known tools used for troubleshooting performance
?
flamegraph-rs or samply to generate flamegraph and detect bottleneck overall
perf (linux only) or uprof for amd chips is used to deep dive into low level metrics such as l1,l2 cache hit/miss, branching, tlb etc


#flashcards/systemengineering