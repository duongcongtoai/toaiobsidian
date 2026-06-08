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

Distinct difference from two implementation
### Per batch data store
```rust
struct ArrayAggGroupsAccumulator {
    datatype: DataType,
    ignore_nulls: bool,
    /// Source arrays — input arrays (from update_batch) or list backing
    /// arrays (from merge_batch).
    batches: Vec<ArrayRef>,
    /// Per-batch list of (group_idx, row_idx) pairs.
    batch_entries: Vec<Vec<(u32, u32)>>,
    /// Total number of groups tracked.
    num_groups: usize,
}
```
Indexing new batch into this data structure is append only fashion
```
        for (row_idx, &group_idx) in group_indices.iter().enumerate() {
            // Skip filtered rows
            if let Some(filter) = opt_filter
                && (filter.is_null(row_idx) || !filter.value(row_idx))
            {
                continue;
            }

            // Skip null values when ignore_nulls is set
            if let Some(ref nulls) = nulls
                && nulls.is_null(row_idx)
            {
                continue;
            }

            entries.push((group_idx as u32, row_idx as u32));
        }

        // We only need to record the batch if it was non-empty.
        if !entries.is_empty() {
            self.batches.push(Arc::clone(input));
            self.batch_entries.push(entries);
        }
```
with a trade-off at the time of reading the batch data on compute/aggregation time
```
        let flat_values = if total_rows == 0 {
            new_empty_array(&self.datatype)
        } else {
            let mut interleave_indices = vec![(0usize, 0usize); total_rows];
            for (batch_idx, entries) in self.batch_entries.iter().enumerate() {
                for &(g, r) in entries {
                    let g = g as usize;
                    if g < emit_groups {
                        let wp = write_positions[g] as usize;
                        interleave_indices[wp] = (batch_idx, r as usize);
                        write_positions[g] += 1;
                    }
                }
            }

            let sources: Vec<&dyn Array> =
                self.batches.iter().map(|b| b.as_ref()).collect();
            arrow::compute::interleave(&sources, &interleave_indices)?
        };
```
Notice the write to write_positions can be random as well, but it max == max group cardinality per emission time (worst case when emit_all)

Compared to organize batch by group
```rust
#[derive(Debug)]
struct ArrayAggGroupsAccumulatorPerGroup {
    datatype: DataType,
    ignore_nulls: bool,
    /// Input batches received via `update_batch`.
    batches: Vec<ArrayRef>,
    /// Per-group list of `(batch_index, row_index)` pairs into `batches`.
    indices: Vec<Vec<(u32, u32)>>,
    /// Number of index entries referencing each batch.
    batch_refcounts: Vec<u32>,
    /// Per-group array chunks from `merge_batch`.
    merged: Vec<Vec<ArrayRef>>,
}
```
at update_batch time the l2 cache can be polluted by random group_idx jumping
```

        for (row_idx, &group_idx) in group_indices.iter().enumerate() {
            // Skip filtered rows
            if let Some(filter) = opt_filter {
                if filter.is_null(row_idx) || !filter.value(row_idx) {
                    continue;
                }
            }

            // Skip null values when ignore_nulls is set
            if let Some(ref nulls) = nulls {
                if nulls.is_null(row_idx) {
                    continue;
                }
            }

            if !batch_pushed {
                self.batches.push(Arc::clone(input));
                self.batch_refcounts.push(0);
                batch_pushed = true;
            }
            self.batch_refcounts[batch_idx] += 1;
            self.indices[group_idx].push((batch_idx as u32, row_idx as u32));
        }
```
This random jump harms more than the previous's implementation's jump

### Per function micro analysis
#### update_batch


## Related Notes & Tasks
- This is a real-world case study for the concepts being tested in **[[Implement tight loop benchmark]]**.
- Relevant hardware concept: **[[Hardware Prefetcher]]** and **[[Cache Pollution]]**.