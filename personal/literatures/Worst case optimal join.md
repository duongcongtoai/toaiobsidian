# Worst case optimal join
Created: 2023-10-23
Given a query 
```
....
from r,s,t where r.a=t.a and r.b=s.b and s.c=t.c

```
The tries built from those 3 tables look like
![[Pasted image 20231023214055.png]]
Let's break down

consider r.a = t.a first, start with 0 from r.a value, probe t we have 3 matching rows:
00
01
02
now probe s we have 3 matching rows
00
01
02
how use condition s.c =t.c to find the intersection, we have
a,b,c
000
001
002

continue with the next value (sorted) in r 0,1

probe t we have 00,01,02 again
probe s we have 10 only
now intersect t.c with s.c: (0,1,2) with (0)
matching row: 0,1,0
... and so on

TODO: multi-way hash trie join







## References
1. https://www.youtube.com/watch?v=knyIDmbJQno&list=PLSE8ODhjZXjYzlLMbX3cR0sxWnRM7CLFn&index=14 
tags: 