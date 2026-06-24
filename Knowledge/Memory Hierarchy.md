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
This note details the CPU memory hierarchy (L1, L2, L3 caches vs. RAM) and the concept of Cache Lines. It explains why reading contiguous memory blocks (like 64-byte lines) is faster and more efficient than random access.

# Memory Hierarchy & Cache Lines

The CPU is magnitudes faster than standard RAM. Caches exist to bridge this latency gap.

- **L1 Cache:** Tiny (e.g., 32 KiB per core) but hyper-fast (~1-2 cycles latency).
- **L2 Cache:** Medium (e.g., 1 MiB per core), slightly slower (~10-15 cycles latency).
- **L3 Cache:** Large (e.g., 16+ MiB shared), slower (~40-75 cycles latency).
- **RAM:** Massive, but extremely slow (200-300+ cycles latency).

## The 64-Byte Cache Line

When a CPU fetches data from RAM, it doesn't fetch a single variable. Instead, it reads a **64-byte block** known as a cache line.
This hardware trade-off minimizes request overhead while avoiding wasting bandwidth on useless data.

For example, a single 64-byte cache line holds exactly 8 `i64` (64-bit) integers.

Efficient algorithms, such as those relying on [[Hardware Prefetcher]]s or vectorized loops, ensure that memory reads are contiguous, maximizing cache line utilization and keeping the execution pipeline full. Random access patterns (like large hash tables) lead to [[Cache Pollution]] and stall the CPU.
---
## Flashcards

What is the typical latency difference between L1 Cache and Main RAM?
?
L1 Cache is extremely fast (~1-2 cycles latency), whereas Main RAM is massive but very slow (200-300+ cycles latency).
<!--SR:!2026-06-24,2,210-->

What is a Cache Line and what is its standard size?
?
A cache line is the fixed block of data the CPU fetches from RAM at once. Its standard size is 64 bytes (which can hold exactly eight 64-bit integers).
<!--SR:!2026-06-27,5,230-->

Why does random memory access stall the CPU compared to contiguous access?
?
Random access breaks the Hardware Prefetcher's pattern recognition and leads to Cache Pollution. The CPU is forced to wait hundreds of cycles for RAM instead of utilizing pre-fetched contiguous cache lines.
<!--SR:!2026-06-25,3,230-->

#flashcards/systemengineering
