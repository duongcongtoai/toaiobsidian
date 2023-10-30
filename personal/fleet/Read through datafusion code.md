# Read through datafusion code
Created: 2023-10-22
## fn create_physical_plan
convert logical plan to physical plan
is a method from SessionState

optimize logical plan then call create_physical_plan over the logical plan
[[QueryPlaner Create Physical Plan]]
## Understand ExecutionPlan trait and its method

## SendableRecordBatchStream

Result of an exectionPlan
an async stream (program can cooperatively read result from this object, meaning if the reader of this obj is managed by an async runtime, it is allow to sleep if there is no result available at the moment, return (cooperatively) control to the runtime so it can do other works that are doable)
Every executionplan  do the same calculation on some number of partition. Partition represent a distinct stream of values
## SymmetricHashJoinExec
## HashJoinExec

## References
1. 
tags: 