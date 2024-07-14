# OLAP encoding scheme
Created: 2024-07-13
## RLE
## Delta encoding

Instead of storing the whole value, only store the deltas between continous value
=> to represents 0,1,2,3 we only use 0,1,1,1
This hurt SIMD, which can do multiple instruction at a time

=> Split the set of values into multiple delta sequence, for example:
a values of 100 items are splitted into 4 chunks
0->24,25->49,50->74,75->99
and they are stored like this in a sequence
These are the indices of the records: 0,25,50,75,1,26,51,76,2,27,52,77,....

SIMD operation happens for 4 integers at a time and in batch operation like:
```
values[1] = values[0] + delta[0]
values[26] = values[25] + delta[25]
values[51]  = values[50] + delta[50]
values[76]  = values[75] + delta[75]
....
```

Because the memory of values `[1,26,51,76]` and `delta[0,25,50,75]` are continous, we can have SIMD support with this storage layout
## Frequency encoding
## Bit-slice encoding
For each bit kth in the values, create a separate columns to determine if the bit kth of the value is set or not. 
![[Pasted image 20240714120116.png]]

## Pseudodecimals
## FSST
## Roaring bitmap

Given a universe of integers, roaring bitmap divide this universe into chunks (each hold 2^16 values), to determine the chunk a value k belongs to, do k/2^16.

If number of values inside a chunk < 4096, store them as is (array of values)
We know that inside each chunk, the value will never reach > 2^16 , thus the maximum space required to store all possible number of values are (16 bits x 4096=8192 byte)

If number of values inside a chunk > 4096, store them as bitmap, to determine which bit represent an integer, do modulo k%2^16 , thus the space requires by a large bitmap to represent 2^16 integers are 8192 byte
## FOR + bitpacking
## References
1. https://www.youtube.com/watch?v=h3EOs7GH7hM 
2. https://github.com/sundy-li/strawboat?tab=readme-ov-file
3. https://www.cs.cit.tum.de/fileadmin/w00cfj/dis/papers/btrblocks.pdf
tags: #olap #encoding
