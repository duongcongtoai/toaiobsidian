Hyper paper describes a query scheduler that divides workload into a fixed sized set of tuples. 
This approach decides the amount of thread to allocate to a task dynamically, and make decisions based on data locality (incase of NUMA, the node that has local copy of the intermediate result used to execute the task)
Morsel-Driven Parallelism: A NUMA-Aware Query Evaluation Framework for the Many-Core Age: https://15721.courses.cs.cmu.edu/spring2023/papers/07-scheduling/p743-leis.pdf

This still has several limitation
- scheduler does not consider the complexity of the task described by the query plan (for example a predicate of a simple matching will cost less than a string matching)
- a short running task can be blocked by a long running one => did not consider the priority

=> here comes umbra: Self-Tuning Query Scheduling for Analytical Workloads https://15721.courses.cs.cmu.edu/spring2023/papers/07-scheduling/wagner-sigmod21.pdf 
- Task not created statically at runtime
- A task can contain muliple morsel
- Modern implementation of stride scheduling?
- Priority decay: aka, when a query is first created, it is assumed to be short => highest priority, overtime if it takes longer, its priority is decayed exponentially

The notion of unit of work no longer be a morsel, but a time duration

### Stride scheduling
pull based approach 
each worker mantains a local thread metadata about which tasks to execute
a global task set slots
\[query1task1,query2task1\]

each slot is a pointer to a sequence of task (task sets), for example query1task1 pointer points to the set of 
\[task1,task2,task3\]
Note that they are dependency graphs, not an straight structure like array

Now in local of each thread:
- active slots: a bitmap/array indicating which entries in the global slot has active task to execute
- change mask: indicates when a new task is added to the global task set array
- return mask: indicates that a taskset in the global task slot has be executed by some of other threads. 

All of these 3 are bitmap, the reason is thread can CAS conveniently to broadcast change to other thread 

Example:
taskset:s ts1,ts2
thread1: executing ts1
thread2: doing unrelated stuff

thread1 complete ts1
thread1 update global task slot, change query1ts1 to iquery1ts2 (cas)
thread1 goes to each of other threads's return mask and CAS to broadcast this event (i've done something, you now check and see what you can further do)

tag:#queryscheduling


