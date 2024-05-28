
# Predicate pruning techniques in OLAP system
Created: 2024-05-28

## Page pruning
Page index (aka page metadata)

Footer (page level) maintains a vector of page indices (TODO: not sure if it's page level or row group level)
each index maintain statistic of min_max
if a predicate prune out a set of rows using one index, corresponding rows from other column should also be pruned out 
e.g
Column1
offset 0
num_rows_in_row_group: 100
pruned out rows: 50-> 100

Then for Column2
offset 100
num_rows_in_row_group: 1000
pruned out rows
TODO
read datafusion code, keyword is PagePruningPredicate
## Row group pruning
Analogous to partition pruning
Each row group maintain statistic such as min-max, certain predicate like value \<\>\= can benefit from this statistics 
However, if the data is pseudo random (uuids), it is not efficient to maintain these datas, in which case bloom filter pruning can be applied

## Bloom filter
![[Pasted image 20240528110711.png]]
In [[Parquet]] bloom filter is maintained per ColumnChunk, quickly tell if a value exist in this chunk

## References
1. https://www.influxdata.com/blog/querying-parquet-millisecond-latency/
tags: #datafusion #olap #predicatepruning 

