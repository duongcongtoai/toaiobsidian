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

## Physical Separation & VIPT (Virtually Indexed, Physically Tagged)

It is crucial to understand that the **L1 TLB is a completely separate hardware component from the L1 Data Cache (L1d) or L1 Instruction Cache (L1i)**. 

While they sit physically adjacent on the silicon die, they store different things: the L1 Cache stores the actual data/instructions, while the L1 TLB stores the address maps. 

They operate in parallel via a mechanism called VIPT:
1. The CPU sends the Virtual Address to both components simultaneously.
2. The L1 Cache uses the virtual address to start indexing and finding the data block.
3. Concurrently, the L1 TLB translates the virtual address into a physical address tag.
4. The TLB provides this physical tag to the L1 Cache at the exact moment the cache finishes its lookup, allowing the CPU to confirm a "hit" and retrieve the data in just 1-2 clock cycles.

---
## Flashcards

What are the L1 and L2 Translation Lookaside Buffers (TLB)?
?
The TLB caches the translation from Virtual Memory Addresses to Physical RAM Addresses.
- **L1 TLB:** Tiny (e.g., 64 entries) and instant (~1 cycle latency).
- **L2 TLB:** Larger (e.g., 1,500 entries) and slightly slower.
<!--SR:!2026-06-24,2,210-->

Is the L1 TLB just a dedicated section of the L1 Data Cache, or is it a separate component?
?
It is a completely separate hardware component. The L1 Cache stores actual data, while the L1 TLB stores virtual-to-physical address maps. They operate in parallel (VIPT) so the TLB can provide the physical tag at the exact moment the L1 Cache finishes its virtual index lookup.
<!--SR:!2026-06-24,2,210-->

#flashcards/systemengineering