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
This note defines the Translation Lookaside Buffer (TLB), an essential CPU cache used to translate virtual memory addresses to physical addresses, and explains the impact of TLB Misses.

# Translation Lookaside Buffer (TLB)

Memory is divided into **4 KiB Pages**. When the CPU requests data via a Virtual Address, it must translate this into a Physical Address located in RAM. Reading the OS Page Table from RAM is slow. The **Translation Lookaside Buffer (TLB)** acts as a cache for this mapping.

- **L1 TLB:** Tiny (e.g., 64 entries) and instant (~1 cycle).
- **L2 TLB:** Larger (e.g., 1,500 entries) and slightly slower.

## TLB Misses and Page Walks

If an address translation misses both L1 and L2 TLBs, the CPU stalls and must perform a **"Page Walk"** to RAM to find the mapping, which is highly expensive.

In scenarios involving large random access patterns—such as querying massive hash tables exceeding L2 cache size—the CPU frequently accesses memory across tens of thousands of pages. This leads to **TLB Thrashing**, where the CPU suffers a TLB miss followed immediately by a Data Cache miss, crippling performance.