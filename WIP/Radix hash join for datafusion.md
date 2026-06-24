---
type: wip
status: drafting
date: 2026-06-24
topic:
tags:
  - wip
ai-first: true
---

## For future Claude
> **AGENT DIRECTIVE:** This is a Work In Progress (WIP) document used as a human-first sandbox. It contains unfinished coding demos, open questions, or unstructured research drafts. **Do not treat the contents of this file as verified ground-truth knowledge during general vault queries.** This file's contents should only be synthesized into the permanent Knowledge base when the user explicitly runs the `/obsidian-wip-graduate` command.

## Context / Draft
implement this https://github.com/apache/datafusion/issues/18939
## What is Radix Hash Join?
How it works:
A traditional Hash Join takes the smaller table (the "build" side) and creates one massive hash table in memory, which the "probe" side then scans against. 
A Radix Hash Join is a two-phase algorithm designed to be highly aware of CPU architecture:
2. Partitioning Phase: Instead of building a single hash table, it takes both the build and probe relations and partitions them into many smaller "clusters" using the lower bits (the radix) of the join key's hash. 
3. Join Phase: Once the data is clustered, it builds a hash table for each partition independently and joins the corresponding build and probe partitions.

## Why current hash join is slow
![[Pasted image 20260624153239.png]]
Execution is mostly inside this function `get_matched_indices_with_limit_offset`
current hash table structure
a hash map between hash value and the first entry in the linked list
```
map<hash,lastentry>
[,,,,,,,,,,first entry,,,,,second last entry]
```

(TODO: see how ArrayMap works and how does the probing works that caused this bottleneck)
### How many tables does it construct?
It depends on the execution plan:
- CollectLeft mode: It broadcasts the entire build side to all worker threads and constructs exactly 1 shared hash table.
- Partitioned mode: The build side data is distributed across multiple streams. Each concurrent partition constructs its own independent hash table for its chunk of data. Thus, it constructs $N$ hash tables, where $N$ is the number of active CPU partitions.
Is the size of the table friendly to the CPU's cache?
DataFusion's developers have optimized the data structures heavily for cache locality, but the overall algorithm is not cache-friendly for large datasets. 
## Is the size of the table friendly to the CPU's cache?

DataFusion's developers have optimized the data structures heavily for cache locality, but the overall algorithm is not cache-friendly for large datasets. 
The Good (Structure):
- Instead of doing thousands of small heap allocations for chained lists, DataFusion uses a single flat next array.
- If the table has fewer than 4.2 billion rows, it dynamically initializes JoinHashMapU32 instead of u64. Using 32-bit integers halves the memory footprint of the index, fitting twice as much data into the CPU cache.
- The hashbrown implementation uses SIMD instructions to probe 16 bucket control bytes simultaneously.
The Bad (Algorithm):
Despite having optimized internal structures, if the build side is 10 GBs, the hash table is going to be far larger than the CPU's ~32MB L3 cache. Just like the issue states, probing into this giant flat array causes random memory access patterns (chain_traverse). This is exactly why the author of issue #18939 is proposing the Radix Hash Join—to chunk that massive table down into cache-sized, bite-sized pieces.
Paper: https://15721.courses.cs.cmu.edu/spring2016/papers/balkesen-icde2013.pdf

https://db.in.tum.de/~bandle/papers/bandle-partitionVsNonPartition.pdf