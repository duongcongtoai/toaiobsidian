---
type: concept
date: 2026-06-07
tags:
  - concept
  - hardware
  - performance
  - cpu
sources:
  - "[[raw/articles/2026-06-07 — Hardware-Level Performance and DataFusion Aggregation.md]]"
ai-first: true
---
## For future Claude
This note describes Hardware Prefetchers, specialized CPU components that predict memory access patterns and preload data from RAM into L1/L2 caches to hide memory latency.

# Hardware Prefetchers

To prevent the CPU's Execution Unit from stalling due to slow RAM access, modern CPUs use **Hardware Prefetchers**. These are dedicated silicon units operating in a dual-pipeline:

1. **L2 Prefetcher (The Aggressive Scout):** Monitors memory requests from L1. When it detects a sequential access pattern (or specific strides), it pulls data from RAM into the 1 MiB L2 cache well ahead of time. This hides the ~250-cycle RAM latency.
2. **L1 Prefetcher (The Just-In-Time Courier):** Watches the Execution Unit. While current data is being processed, it requests the *next* block of data from L2 into the tiny L1 cache. Because the L2 prefetcher already loaded it from RAM, this is a fast L2 cache hit.

## The Page Boundary Wall

Prefetchers will stop aggressively fetching when they hit a **4 KiB Page Boundary** if the next page's address isn't in the [[Translation Lookaside Buffer]] (TLB). This behavior prevents the prefetcher from triggering OS Page Faults by guessing wrong. Once the program execution crosses the boundary and updates the TLB, prefetching resumes.

---
