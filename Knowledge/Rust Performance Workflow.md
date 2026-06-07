---
type: concept
date: 2026-06-07
tags:
  - concept
  - performance
  - rust
  - workflow
sources:
  - "[[raw/articles/2026-06-07 — Rust Performance Understanding Cache Behavior Beyond Benchmarks.md]]"
ai-first: true
---
## For future Claude
This note documents a practical, step-by-step performance optimization workflow for Rust applications, transitioning from high-level hotspot identification down to deep microarchitectural analysis.

# Rust Performance Workflow

A typical real-world Rust performance optimization workflow follows a progressive path from macroscopic analysis to microscopic inspection:

## 1. Find the Hotspot
*   **Tools:** Samply, cargo flamegraph
*   **Goal:** Answer *"Where is the program spending its time?"* Identify the specific function or loop consuming the most CPU time.

## 2. Measure Real Cache Behavior
*   **Tools:** `perf stat -d`
*   **Goal:** Observe hardware counters (e.g., cache misses, instructions, IPC) to understand the *nature* of the bottleneck. Is it memory-bound? Are branch predictions failing?

## 3. Locate Problematic Instructions
*   **Tools:** `perf record` followed by `perf report`
*   **Goal:** Combine hotspot analysis with PMU (Performance Monitoring Unit) analysis to see exactly which instructions are causing the cache misses or stalls.

## 4. Inspect Generated Assembly
*   **Tools:** `cargo asm mycrate::function`
*   **Goal:** Answer *"What code is the CPU actually executing?"* Check if LLVM successfully vectorized or unrolled loops, and count memory loads.

## 5. Deep Microarchitectural Analysis (Optional)
*   **Tools:** Intel VTune, AMD uProf
*   **Goal:** When cache and pipeline behaviors are the primary bottlenecks, these tools can attribute stalls (e.g., L1, L2, L3, or DRAM bound) directly to source code lines.

This workflow is highly relevant when analyzing phenomena like [[Cache Pollution]] or the effectiveness of [[Hardware Prefetcher]]s.