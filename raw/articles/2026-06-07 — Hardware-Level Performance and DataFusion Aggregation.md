---
date: 2026-06-07
tags:
  - source
  - article
source_type: article
ai-first: true
---
## For future Claude
This article explains the hardware-level performance implications for database engines like DataFusion. It covers the memory hierarchy, cache lines, hardware prefetching, TLB page boundaries, and how high-cardinality grouping leads to cache/TLB thrashing, which justifies DataFusion's adaptive skipping for partial aggregation.

# Research Notes: Hardware-Level Performance & DataFusion Aggregation

## 1. The Memory Hierarchy & Cache Lines

To understand database performance, we must understand that the CPU is extremely fast, but RAM is extremely slow. Caches bridge this gap.

- **L1 Cache:** Tiny (e.g., 32 KiB per core) but hyper-fast (~1-2 cycles).
- **L2 Cache:** Medium (e.g., 1 MiB per core), slightly slower (~10-15 cycles).
- **L3 Cache:** Large (e.g., 16+ MiB shared), slower (~40-75 cycles).
- **RAM:** Massive, but extremely slow (200-300+ cycles).

**The 64-Byte Cache Line:**
When a CPU reads from RAM, it does not read a single integer. It reads a **64-byte block** called a cache line.

- *Why 64 bytes?* It is a hardware trade-off. Smaller would cripple bandwidth with request overhead. Larger would waste memory bandwidth by dragging useless bytes across the motherboard during random access.
- *Example:* If we read 64-bit integers (`i64`), one integer is 8 bytes. Therefore, a single 64-byte cache line holds exactly **8 integers**.

## 2. Performance Gain from Tight-Loop (Predictable Access)

When DataFusion scans a dense array (like a batch of 8,192 rows), it processes them sequentially via heavily vectorized tight loops. This allows the CPU's memory hierarchy to function perfectly.

- **How the Execution Unit Works (The L1 Gateway):**
While processing a cache line (e.g., Rows 0-7), the CPU executes `LOAD` instructions. These instructions *must* pull 8-byte chunks from the L1 Data Cache directly into CPU hardware registers. The CPU execution unit cannot bypass L1 to read from L2 or RAM directly for standard loads. The ALU then performs computations on those registers. Because the data is in L1, these loads take only ~1 cycle.
- **The Magic of the Dual-Prefetcher Pipeline:**
To ensure the Execution Unit never runs out of data in the L1 cache, dedicated silicon chips called **Hardware Prefetchers** work in a coordinated two-stage pipeline:
    1. **L2 Prefetcher (The Aggressive Scout):** It monitors the stream of memory requests coming out of L1. Upon detecting a sequential pattern, it looks far into the future (e.g., 10-20 cache lines ahead). It absorbs the 250-cycle RAM latency by fetching large blocks of data from RAM into the 1 MiB L2 cache to prepare for the potential future workload of L1.
    2. **L1 Prefetcher (The Just-In-Time Courier):** It watches the Execution Unit directly. While the CPU is executing and loading Rows 0-7 from the L1 cache, the L1 prefetcher predicts the need for the *next* batch of rows (Rows 8-15). It sends a request to the L2 Cache to pull that specific 64-byte block into the tiny 32 KiB L1 cache.
    3. **The Guarantee:** Because the aggressive L2 Prefetcher has already staged the upcoming data from RAM, the L1 Prefetcher's request to L2 is a guaranteed "Cache Hit". The next 64-byte block is moved from L2 to L1 in ~10 cycles, arriving *before* the CPU even asks for Row 8.
- **The Result:** This pipeline ensures that data is always sitting in the L1 cache exactly when the CPU registers demand it. The tight, branchless loops in DataFusion (like those in `create_hashes`) allow this pipeline to scan data at the absolute physical limit of the RAM's bandwidth.

### The Role of the TLB (Translation Lookaside Buffer) and Page Boundaries

The interaction between the Hardware Prefetcher and memory also heavily involves the TLB. Memory is divided into **4 KiB Pages**, and the TLB translates virtual addresses to physical addresses page-by-page.

- **The "Page Boundary" Wall:** As the prefetcher eagerly runs ahead fetching cache lines, it will **stop** when it hits a 4 KiB page boundary if the next page's physical address is not already cached in the TLB.
- **Why it stops:** A TLB miss means the CPU has to do a "Page Walk" (reading OS memory maps). The prefetcher won't trigger this blindly to avoid triggering OS Page Faults for data you might not actually need.
- **Resuming:** The prefetcher waits for your actual code to cross the boundary, suffer the TLB miss legitimately, and update the TLB. Once the TLB has the new translation, the prefetcher wakes back up and rockets through the next 4 KiB. Because 4 KiB holds a lot of data (e.g., 512 integers), this hiccup is incredibly rare compared to the random-access nightmare of a massive hash table!

## 3. Performance Gained for L2-Sized Compacted Hash Tables

If the prefetcher handles sequential arrays, how does a hash table get into the cache? It uses **Demand Fetching**.

- **Example: Grouping by `City` (10,000 distinct values).**
A 10,000-entry hash table might take ~160 KiB of RAM. This is too big for the 32 KiB L1 cache, but fits perfectly inside the 1 MiB L2 cache.
- **The "Cold" Start:**
When DataFusion hashes the first row ("New York") and jumps to `Bucket 4,021`, the CPU misses all caches and demands the 64-byte chunk from RAM. It suffers a 250-cycle penalty. It does this for the first few thousand rows.
- **The "Warm" State:**
Because the entire 160 KiB hash table fits inside the L2 cache, the cache lines are never evicted! After the first few thousand rows, every random hash jump results in an **L2 Cache Hit** (~10 cycles). The database hums along at blazing speeds.

## 4. The High-Cardinality Trap (Cache Pollution)

The system falls apart when grouping by high-cardinality columns (e.g., `Transaction_ID` with 50 million unique values).

- A 50-million-entry hash table takes hundreds of megabytes. It cannot fit in L1, L2, or even L3.
- **Cache Thrashing / Pollution:**
    1. Row 1 hashes to `Bucket 1,000,000`. Fetched from RAM.
    2. Row 2 hashes to `Bucket 5`. Fetched from RAM.
    3. Row 3 hashes to `Bucket 40,000,000`. Fetched from RAM.
- Because the table is larger than the cache, the CPU constantly evicts old buckets to load new ones. The table never gets "warm." The prefetcher is intentionally disabled by the hardware because the access pattern is completely random.
- **TLB Thrashing (The Double Penalty):** High cardinality causes not just cache misses, but **TLB misses**. A 500 MB hash table spans 128,000 pages, far exceeding the TLB size (e.g., 1,500 entries). When the CPU jumps randomly to a new bucket, it suffers a TLB Miss (requiring a Page Table Walk to find the physical address) *and then* a Cache Miss (requiring a RAM fetch for the data). This double-whammy is called thrashing.
- **The Result:** The CPU spends 90% of its time stalled, waiting 250+ cycles for RAM on every single row.

## 5. Conclusion: Why DataFusion Needs Adaptive Skipping (Issue #22405)

This hardware reality explains why DataFusion's **Partial Aggregation** must be skipped in high-cardinality scenarios.

- Building a local hash table is designed to reduce the volume of data shuffled across the network.
- If cardinality is low, the hash table fits in L2 (warm cache), runs instantly, and compresses the network shuffle.
- If cardinality is high, the hash table ruins CPU performance via cache pollution, and provides zero network compression (since every row is unique anyway).
- **The Fix:** The adaptive cost model (measuring `partial_ns_per_row`) acts as a tripwire. When cache pollution starts happening (detected by a massive spike in nanoseconds-per-row due to RAM stalls), DataFusion aborts the hash table entirely and falls back to directly streaming the raw rows.

---

# Take-Home Exercises

1. **Implement a code block that proves a tight loop outperforms a non-tight loop.***(Goal: Write a small Rust or C program that compares iterating over an array contiguously versus a loop that contains heavy branching/function calls preventing vectorization).*
2. **Prove that L2-sized hashtables outperform large hash tables.***(Goal: Write a benchmark script that allocates a 100KB hash table and a 1GB hash table, does 10 million random lookups in both, and compares the throughput/latency to directly observe cache misses in action).*
3. **Simulate TLB Pollution Causing Performance Degradation.***(Goal: Write a C/Rust benchmark that allocates a large array spanning tens of thousands of pages. Compare the performance of accessing elements sequentially (where the TLB handles 4 KiB chunks smoothly) versus jumping randomly to a new 4 KiB page on every single access (guaranteeing a TLB miss on every read). This will isolate and demonstrate the extreme penalty of TLB thrashing).*

## 6. Address Translation & The TLB Hierarchy

Memory is divided into **4 KiB Pages**. When the CPU requests data, it uses a Virtual Address, which must be translated into a Physical Address in RAM. The OS Page Table holds this map, but reading it from RAM is very slow. To solve this, the CPU uses a **Translation Lookaside Buffer (TLB)**.

- **L1 TLB:** Tiny (e.g., 64 entries) and instant (~1 cycle).
- **L2 TLB:** Larger (e.g., 1,500 entries) and slightly slower, acting as a backup.
- **TLB Miss & Page Walk:** If an address translation misses both L1 and L2 TLBs, the CPU stalls and performs a "Page Walk" to RAM to find the mapping, which is extremely expensive.

**TLB Thrashing in DataFusion:**
When DataFusion builds a massive 500 MB hash table for high-cardinality grouping, the table spans roughly 128,000 memory pages. Randomly jumping across 128,000 pages completely overwhelms the ~1,500-entry L2 TLB. The CPU suffers from both Data Cache Misses *and* TLB Misses simultaneously (Thrashing), causing performance to plummet.

## 7. Multi-Level Prefetching (L1 vs L2)

Prefetching happens at multiple levels in the cache hierarchy:

- **L2 Prefetcher:** Acts as a massive staging area. It watches memory requests and aggressively pulls large blocks of data from RAM into the 1 MiB L2 cache using algorithms like "Next-Line" and "Stride" (detecting if you are skipping by fixed intervals). It hides the 250-cycle RAM latency.
- **L1 Prefetcher:** Focuses on immediate needs, pulling data from the L2 cache into the tiny 32 KiB L1 cache just in time for the CPU.

**The Page Boundary Wall:** The hardware prefetcher will purposefully stop when it hits a 4 KiB page boundary if the translation for the next page is not already in the TLB. This prevents it from triggering OS Page Faults by guessing wrong. Once your code legitimately crosses the boundary and updates the TLB, the prefetcher resumes. In random-access workloads (like massive hash tables), the prefetcher shuts itself down entirely to avoid wasting memory bandwidth.