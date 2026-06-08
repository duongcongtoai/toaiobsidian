---
date: 2026-06-07
tags:
  - source
  - article
source_type: article
ai-first: true
---
## For future Claude
This raw source outlines how to understand CPU cache behavior in Rust beyond simple benchmarks. It contrasts software simulation with hardware measurement using CPU Performance Monitoring Units (PMUs) and details a practical Rust performance workflow using various profiling tools.

# Rust Performance: Understanding Cache Behavior Beyond Benchmarks

## Goal

Learn how CPU caches (L1/L2/L3), memory access patterns, and tight loops affect the performance of Rust code.

A key distinction is understanding the difference between **simulation** and **measurement**.

---

# Simulation vs Measurement

## Simulation

Simulation predicts how a cache hierarchy would behave using a software model.

Tools:

* iai-callgrind
* Valgrind Cachegrind
* gem5
* Sniper

Example output:

```text
L1 D-cache misses: 12345
LL cache misses: 678
```

These values are generated from a cache model and are **not actual hardware measurements**.

### Advantages

* Deterministic
* Reproducible
* No measurement noise
* Excellent for comparing two implementations

### Disadvantages

Cannot accurately model:

* Hardware prefetchers
* Speculative execution
* Cache replacement policies
* NUMA effects
* SMT interactions
* Vendor-specific CPU behavior

Therefore:

> Cachegrind answers "What would happen under this cache model?"

not

> "What actually happened on my CPU?"

---

## Hardware Measurement

Hardware measurement uses the CPU's Performance Monitoring Unit (PMU).

Tools:

* perf
* Intel VTune
* AMD uProf

Example counters:

```text
L1 cache misses
L2 cache misses
L3 cache misses
Branch mispredictions
Cycles
Instructions retired
IPC
```

These are measurements collected from the actual CPU.

---

# Where Samply Fits

Samply is a sampling profiler.

Good for:

* Finding hot functions
* Flame graphs
* Thread analysis
* CPU time distribution

Not designed for:

* L1 cache misses
* L2 cache misses
* Branch mispredictions
* CPI analysis

Think of Samply as answering:

> "Where is the program spending time?"

rather than

> "Why is the CPU stalled?"

---

# Real-World Rust Performance Workflow

Most Rust performance work follows this progression:

```text
Samply / Flamegraph
        ↓
Find hotspot
        ↓
perf stat
        ↓
Measure cache behavior
        ↓
perf record
        ↓
Locate problematic instructions
        ↓
cargo asm
        ↓
Inspect generated assembly
        ↓
VTune (optional)
        ↓
Deep microarchitectural analysis
```

---

# Tool Comparison

| Tool             | Purpose                    | Real Hardware? |
| ---------------- | -------------------------- | -------------- |
| cargo flamegraph | Find hotspots              | Yes            |
| Samply           | Sampling profiler          | Yes            |
| perf stat        | Hardware counters          | Yes            |
| perf record      | Hotspot + PMU analysis     | Yes            |
| cargo asm        | Inspect generated assembly | N/A            |
| iai-callgrind    | Simulated cache analysis   | No             |
| Cachegrind       | Simulated cache analysis   | No             |
| Intel VTune      | Deep CPU analysis          | Yes            |
| AMD uProf        | Deep CPU analysis          | Yes            |

---

# Recommended Learning Path

## Step 1: Understand Generated Code

Inspect LLVM output:

```bash
cargo asm mycrate::function
```

Questions:

* Was the loop vectorized?
* Was it unrolled?
* How many memory loads occur?

---

## Step 2: Measure Real Cache Behavior

```bash
perf stat -d ./target/release/my_program
```

Interesting counters:

```text
cache-misses
cache-references
cycles
instructions
branches
branch-misses
```

Questions:

* Is the workload memory bound?
* Is branch prediction failing?
* Is IPC lower than expected?

---

## Step 3: Locate Hotspots

```bash
perf record ./target/release/my_program
perf report
```

Questions:

* Which function causes most cache misses?
* Which loop consumes most CPU time?

---

## Step 4: Deep Analysis

Use VTune or uProf.

These tools can answer:

```text
Is the workload:
    L1-bound?
    L2-bound?
    L3-bound?
    DRAM-bound?
```

and often attribute stalls directly to source lines.

---

# Example Experiment

Sequential access:

```rust
for i in 0..n {
    sum += data[i];
}
```

Random access:

```rust
for i in random_indices {
    sum += data[i];
}
```

Measure:

```bash
perf stat -d ./target/release/my_program
```

Observe:

* L1 miss rate
* L2 miss rate
* LLC miss rate

This is one of the simplest ways to build intuition about cache locality.

---

# Practical Takeaway

Use:

```text
Samply
```

to find where time is spent.

Use:

```text
perf
```

to understand why.

Use:

```text
cargo asm
```

to understand what code the CPU is executing.

Use:

```text
VTune/uProf
```

when cache and pipeline behavior become the bottleneck.

For learning CPU cache behavior in Rust, perf remains the most important tool because it measures real hardware events rather than simulated ones.
