---
type: concept
date: 2026-06-07
tags:
  - concept
  - performance
  - cpu
sources:
  - "[[raw/articles/2026-06-07 — CPU Pipeline Utilization and Cache Performance Notes.md]]"
ai-first: true
---
## For future Claude
This note defines CPU pipeline slots and execution cycles, highlighting the mental model that wasted cycles cannot be recovered, which drives the need for optimizations like Out-of-Order execution and SMT.

# CPU Pipeline Slots

In CPU performance analysis, a **pipeline** refers to the instruction execution pipeline. 

Modern superscalar CPUs can issue multiple micro-operations (uops) per clock cycle. Each cycle contains a fixed number of execution opportunities known as **pipeline slots**. 

For example, a 4-wide CPU has 4 slots per cycle:
- Slot 0: ADD
- Slot 1: MUL
- Slot 2: LOAD
- Slot 3: STORE

## The "Train" Mental Model

- **Pipeline Slot** = A seat on a train.
- **Cycle** = The train's departure.

When a cycle passes (the train leaves), any unoccupied slots (empty seats) are lost forever. The CPU cannot "reuse" a wasted cycle later. 

To maximize throughput and ensure as many slots as possible are filled with Retiring (useful) instructions, CPUs employ techniques like [[Out-of-Order Execution]], [[Speculative Execution]], Simultaneous Multi-Threading (SMT), and the [[Hardware Prefetcher]]. These optimizations work in tandem to minimize stalls identified in [[CPU Top-Down Microarchitecture Analysis]].

## Flashcard

#flashcards