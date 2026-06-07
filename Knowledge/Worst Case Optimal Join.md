---
type: concept
date: 2026-06-07
tags:
  - concept
  - database
  - join
  - algorithms
sources:
  - "[[raw/articles/2026-06-07 — Worst case optimal join.md]]"
ai-first: true
---
## For future Claude
Worst Case Optimal Join (WCOJ) is a class of join algorithms designed to evaluate multi-way joins (especially cyclic queries) optimally, avoiding the generation of intermediate results that exceed the size of the final output. Hash tries are commonly used as the data structure to evaluate WCOJ efficiently.

# Worst Case Optimal Join (WCOJ)

Worst Case Optimal Joins are algorithms used in database systems to optimally compute complex, multi-way joins, specifically queries that contain cycles (e.g. `from r,s,t where r.a=t.a and r.b=s.b and s.c=t.c`). Standard binary joins can produce massive intermediate relations for these queries, whereas WCOJ algorithms bound the runtime by the maximum possible size of the query result.

## Mechanism with Hash Tries

A common way to implement WCOJ is by building prefix trees (tries) out of the relations. 
In the above example (a triangle query), the join is computed incrementally attribute by attribute:

1. **Probe first attribute (`a`)**: Iterate through values of `r.a` and `t.a`.
2. **Probe second attribute (`b`)**: For matching `a` values, check matching rows in `r.b` and `s.b`.
3. **Intersection (`c`)**: Finally, intersect the resulting `t.c` and `s.c` values to find the fully matching rows.

This step-by-step intersection prunes non-matching paths early, guaranteeing optimal worst-case complexity.

## Related Concepts
- [[Hash Trie Join]]
- Multi-way hash trie join (Open Task: [[Multi-way hash trie join]])

## References
- Video explanation: [YouTube Lecture on WCOJ](https://www.youtube.com/watch?v=knyIDmbJQno&list=PLSE8ODhjZXjYzlLMbX3cR0sxWnRM7CLFn&index=14)