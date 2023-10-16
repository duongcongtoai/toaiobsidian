## summary
Pull based scheduling with multiple worker threads organized into pools
Each cpu have multiple pools
Each pool has soft and hard priority queue

- Soft means other worker can steal task from it
- Hard means only it is allowed to execute (e.g the task relies on local memory inside its node in a NUMA device)

a separate watchdog thread to check if any pool is saturated and reassign tasks dynamically

### Different pool per group
- working
- inactive: blocked inside the kernel inside a latch (e.g a futex causing the thread to go to sleep)
- free: sleeps for a while, wake up see if there are tasks to execute
- parked: still wait for a task but blocked at kernel level until the watchdog wakes it up (maybe before it went to sleep, it used to be in the state of "free" first)

The insight of maintaining a parked threads pool is because it is cheaper than spawning new thread

NUMA thread executor is pinned on the core of the node, the reason is most of the execution relies on data locality, (in NUMA cross node memory access is expensive). But SAP HANA dynamically detect if a task is CPU bound or memory bound, if it is CPU bound, then in some extent it still allows work-stealing from cross-node threads.

Comparing to Hyper, SAP HANA states that on machine with many more socket, work stealing (thread from this node executing tasks that belong to other node aka cross node execution) is not as beneficial as having the thread work with the data available inside its local memory

Using thread groups allow cores to execute other tasks instead of just queries 

#SAPHANA #queryscheduling #numa  