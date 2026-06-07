---
type: task
date: 2026-06-07
tags:
  - task
  - performance
  - datafusion
status: in-progress
project: Personal
due: TBD
ai-first: true
---
## For future Claude
A task to perform a detailed performance analysis on a micro-optimization in Apache DataFusion's `array_agg` implementation. The optimization changed the memory layout from a per-group organization to a per-batch organization, resulting in a 25x speedup for high-cardinality queries by improving cache locality.

# Analyze array_agg micro-optimization

## Context
Perform a detailed performance analysis comparing two implementations of `array_agg` in DataFusion:
- **Baseline (Per-group organization):** [Commit d303f581](https://github.com/apache/datafusion/blob/d303f5817f696fc6250bf67a71d3ea22b0628124/datafusion/core/benches/aggregate_query_sql.rs)
- **Optimized (Per-batch organization):** [Commit c5238ac4](https://github.com/apache/datafusion/blob/c5238ac439b4fa74cd5e947b22f66ed307cc6d92/datafusion/core/benches/aggregate_query_sql.rs)

See the relevant PR discussion here: [apache/datafusion#20504 (comment)](https://github.com/apache/datafusion/pull/20504#issuecomment-3976212053).

## The Mechanism to Analyze
The initial approach organized aggregate states by group. With high cardinality (many groups), this led to many small allocations and a random memory access pattern, crippling performance due to cache pollution and TLB thrashing. 

The optimized version shifted to a per-batch organization (keeping a reference to batch contents and an array of `(group_idx, row_idx)` pairs). This creates a contiguous memory layout that can be efficiently scanned, allowing the CPU to leverage the **[[Hardware Prefetcher]]**.

## Related Notes & Tasks
- This is a real-world case study for the concepts being tested in **[[Implement tight loop benchmark]]**.
- Relevant hardware concept: **[[Hardware Prefetcher]]** and **[[Cache Pollution]]**.