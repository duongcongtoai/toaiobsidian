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
This note defines Cache Pollution and Cache Thrashing, detailing how high-cardinality operations (like grouping with massive hash tables) overwhelm L1/L2 caches and the TLB, leading to severe CPU stalls.

# Cache Pollution (Thrashing)

Cache Pollution occurs when an operation frequently accesses random memory locations spread across a dataset that is significantly larger than the available CPU caches (L1, L2, L3). 

For example, when a database engine like [[DataFusion]] groups by a high-cardinality column, it builds a massive hash table (e.g., 500 MB). Because the cache can only hold a fraction of this data, random lookups constantly force the CPU to evict old cache lines to load new ones from RAM. The cache never becomes "warm."

## Double Penalty: Data Cache + TLB Thrashing

When the access pattern is completely random across a huge memory space, the system suffers a double penalty:
1. **TLB Miss:** The address translation isn't cached in the [[Translation Lookaside Buffer]], requiring a slow Page Walk.
2. **Data Cache Miss:** The actual data isn't in L1/L2, requiring a ~250-cycle fetch from RAM.

As a result, the CPU spends the vast majority of its time stalled, waiting for memory. This is why DataFusion issue #22405 is trying to employ **Adaptive Skipping** (aborting partial aggregation if `partial_ns_per_row` spikes) to avoid this performance trap.