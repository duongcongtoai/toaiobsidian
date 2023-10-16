https://www.youtube.com/watch?v=pqRV7Mss1Fg&list=PLSE8ODhjZXjYzlLMbX3cR0sxWnRM7CLFn&index=12
## TLB

Program interact with memory using virtual address. When a program tells the os to allocate some chunk of memory, os only gives it a virtual memory at the beginning of that memory region

Translation between virtual add to physical addr using page table

Each entry in page table coresspond to the addr of a page (usually 4kb)

Partition or not:
https://15721.courses.cs.cmu.edu/spring2023/papers/11-hashjoins/schuh-sigmod2016.pdf

https://15721.courses.cs.cmu.edu/spring2023/papers/11-hashjoins/bandle-sigmod21.pdf
Partitioned hash join argues that the cpu cost used for hashing records into the bucket will not outweight the cost of performing a non-partitioned data.

Radix hash join vs normal partitioned hash join
Radix hash join requires extra pass over the input data to have a look at data distribution. For example in the first level of partition, bucket 2 is  skewed (using histogram to track), it can decides to use another level of partition. After that it, it reads the input again to actually do the hashing.
P/S: actually in the first pass, the main thing it does is to assign for each worker thread, for each radix key, what offset it should start writing to
![[Radix partition.png]]
Explain:
Maintain separate prefix sum for each thread:
1st cpu prefix sum: 
0,1,1,0 => 0:2,1:2
2nd cpu prefix sum:
0,1,1,1 => 0:1,1:3
Assigned offsets:
cpu1: 
- radix 0: 0
- radix 1: 3
cpu2:
- radix 0: 2
- radix 1: 5

=> no syncrhonization cost
The result is a sequence of data like partition0:0->11 offset, partition 1: 12->...
Note that if after the 1st pass the number of partition has not reached the desired number of partition, continue with the next radix base (position 2)


### Optimization
- If we are writing to an output buffer, we can use streaming instruction to bypass cpu cache
tag: #ltb #virtualmemory