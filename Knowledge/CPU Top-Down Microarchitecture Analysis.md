---
type: concept
date: 2026-06-07
tags:
  - concept
  - performance
  - cpu
  - microarchitecture
sources:
  - "[[raw/articles/2026-06-07 — CPU Pipeline Utilization and Cache Performance Notes.md]]"
ai-first: true
---
## For future Claude
This note explains the Top-Down Microarchitecture Analysis method, which categorizes CPU execution stalls into five main categories: Retiring, Frontend Bound, Backend Bound, Bad Speculation, and SMT Contention.

# Top-Down Microarchitecture Analysis

The Top-Down methodology is a systematic approach to identifying performance bottlenecks in modern CPUs by categorizing how [[CPU Pipeline Slots]] are utilized or wasted.

## The Five Categories

1. **Retiring (High = Good)**
   - Indicates useful work completed successfully (e.g., arithmetic, successful memory loads/stores).
   - High retiring percentage implies productive CPU utilization.

2. **Frontend Bound (High = Instruction Delivery Bottleneck)**
   - The backend is ready, but the CPU could not fetch or decode instructions fast enough.
   - *Causes:* L1 Instruction Cache misses, ITLB misses, complex instructions, branch prediction fetch stalls.

3. **Backend Bound (High = Memory or Execution Bottleneck)**
   - Instructions are decoded, but cannot execute efficiently.
   - **Memory Bound:** Waiting for data ([[Cache Pollution]], L1/L2/L3 misses, DRAM access).
   - **Core Bound:** Waiting for execution resources (ALU/FPU contention).
   - *Fix:* Ensure contiguous memory access and leverage the [[Hardware Prefetcher]].

4. **Bad Speculation (High = Branch Prediction Problem)**
   - Cycles spent on work that was later discarded due to executing the wrong path.
   - *Causes:* Branch mispredictions, machine clears. This limits the effectiveness of [[Speculative Execution]].

5. **SMT Contention (High = Sibling Interference)**
   - Resource competition between two logical threads sharing one physical core (Hyper-Threading).

## Relationship with Hardware Prefetching
A successful [[Hardware Prefetcher]] reduces **Backend Bound** stalls (by preventing cache misses) and consequently increases the **Retiring** rate. Poor prefetching does the opposite.
---
## Flashcards

In Top-Down Microarchitecture Analysis, what does it mean to be "Frontend Bound"?
?
The execution backend is ready to do work, but the CPU stalled because it could not fetch or decode instructions fast enough (e.g., due to L1i cache misses, ITLB misses, or branch prediction fetch stalls).
<!--SR:!2026-06-24,2,210-->

In Top-Down Analysis, what is the difference between "Memory Bound" and "Core Bound"?
?
Both are subcategories of "Backend Bound" stalls. Memory Bound means the CPU is stalled waiting for data (cache misses, RAM latency). Core Bound means the CPU has the data but is waiting for execution resources (ALU/FPU contention).
<!--SR:!2026-06-24,2,210-->

What does a high "Retiring" percentage indicate in Top-Down Analysis?
?
It is a positive metric indicating that CPU Pipeline Slots were filled with useful work that completed successfully, implying highly productive CPU utilization.
<!--SR:!2026-06-27,5,230-->

#flashcards/systemengineering
