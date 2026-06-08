---
date: 2026-06-07
tags:
  - source
  - article
source_type: article
ai-first: true
---
## For future Claude
This raw source outlines the Top-Down Microarchitecture Analysis methodology, explaining how pipeline utilization works, what pipeline slots are, and the categories of execution stalls (Retiring, Frontend Bound, Backend Bound, Bad Speculation, SMT Contention), as well as optimizations like Out-of-Order execution and hardware prefetching.

# CPU Pipeline Utilization & Cache Performance Notes

## Pipeline in CPU Performance Analysis

In pipeline utilization metrics, a **pipeline** refers to the CPU's instruction execution pipeline, not specifically the L1/L2 cache pipeline.

Modern CPUs can issue multiple micro-operations (uops) per cycle. Each cycle contains a fixed number of execution opportunities called **pipeline slots**.

Example (4-wide CPU):

```text
Cycle N:
Slot 0: ADD
Slot 1: MUL
Slot 2: LOAD
Slot 3: STORE
```

Pipeline analysis tries to answer:

> How were the available pipeline slots used?

---

## Top-Down Pipeline Categories

### Retiring

Useful work completed successfully.

Examples:

* Arithmetic instructions
* Floating-point operations
* Successful loads/stores
* Correctly predicted branches

Interpretation:

* High = good
* Indicates productive CPU utilization

---

### Frontend Bound

The CPU could not supply instructions fast enough.

Common causes:

* L1 Instruction Cache misses
* ITLB misses
* Decode bottlenecks
* Complex instructions
* Branch prediction fetch stalls

Interpretation:

* Backend is ready
* No instructions available to execute

---

### Backend Bound

Instructions are available but cannot execute efficiently.

#### Memory Bound

Waiting for data:

* L1 cache miss
* L2 cache miss
* L3 cache miss
* DRAM access

Example:

```c
sum += array[random_index[i]];
```

#### Core Bound

Waiting for execution resources:

* ALU contention
* FPU contention
* Execution port contention

Interpretation:

* Work exists
* CPU resources or data are unavailable

---

### Bad Speculation

Work executed on the wrong path.

Common causes:

* Branch mispredictions
* Machine clears
* Memory ordering violations

Example:

```c
if (condition)
```

CPU predicts incorrectly, executes the wrong path, then flushes the pipeline.

Interpretation:

* Cycles were spent doing work that was later discarded

---

### SMT Contention

Occurs when Simultaneous Multi-Threading (SMT / Hyper-Threading) is enabled.

Two logical threads share one physical core.

Shared resources:

* Execution ports
* Load/store units
* Cache bandwidth
* Reorder buffer entries

Interpretation:

* One thread reduces resources available to the other

---

## Relationship with Hardware Prefetching

Prefetching is **not** a pipeline category.

Instead, prefetching influences pipeline utilization.

### Good Prefetching

```text
Future data predicted
↓
Data arrives in cache early
↓
Load becomes L1 hit
↓
Retiring increases
```

Metrics:

```text
Retiring ↑
Backend Bound ↓
```

### Poor Prefetching

```text
Load
↓
Cache miss
↓
DRAM access
↓
Long stall
```

Metrics:

```text
Backend Bound ↑
Retiring ↓
```

---

## Does the CPU "Reuse" Cycles?

No.

A cycle that passes is gone forever.

Example:

```text
Cycle N:
Slot 0: Used
Slot 1: Idle
Slot 2: Idle
Slot 3: Idle
```

The idle slots cannot be reused later.

Instead, modern CPUs attempt to **avoid wasting future slots**.

---

## Techniques That Improve Pipeline Utilization

### Out-of-Order Execution (OoO)

Instead of waiting for a slow instruction:

```text
LOAD A (cache miss)
ADD B
MUL C
```

CPU may execute:

```text
ADD B
MUL C
```

while waiting for `LOAD A`.

Goal:

* Fill otherwise idle pipeline slots

---

### Speculative Execution

CPU predicts branch outcomes and executes ahead.

If prediction is correct:

```text
Retiring ↑
```

If incorrect:

```text
Bad Speculation ↑
```

---

### SMT / Hyper-Threading

If Thread A stalls:

```text
Thread A waiting for memory
```

CPU may schedule Thread B:

```text
Pipeline slots occupied by Thread B
```

This is the closest concept to "reusing" otherwise wasted execution opportunities.

---

### Hardware Prefetching

Attempts to ensure future loads hit cache instead of DRAM.

Goal:

```text
Reduce memory stalls
↓
Reduce Backend Bound
↓
Increase Retiring
```

---

## Mental Model

Think of pipeline slots as seats on a train.

```text
Pipeline Slot = Seat
Cycle = Train Departure
```

When the train leaves:

* Occupied seats = useful work
* Empty seats = lost forever

CPU optimizations such as:

* Out-of-Order Execution
* Speculative Execution
* SMT
* Hardware Prefetching

all try to ensure that as many seats as possible are occupied before the train departs.

---

## Quick Interpretation Cheat Sheet

| Metric          | Meaning                              | Desired Direction |
| --------------- | ------------------------------------ | ----------------- |
| Retiring        | Useful work                          | Higher            |
| Frontend Bound  | Waiting for instructions             | Lower             |
| Backend Bound   | Waiting for data/resources           | Lower             |
| Bad Speculation | Wrong-path execution                 | Lower             |
| SMT Contention  | Resource competition between threads | Lower             |

Rule of thumb:

```text
High Retiring = Good
High Backend Bound = Memory or execution bottleneck
High Frontend Bound = Instruction delivery bottleneck
High Bad Speculation = Branch prediction problem
High SMT Contention = Hyper-thread sibling interference
```