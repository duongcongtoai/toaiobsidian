
# Predicate pruning techniques in OLAP system
Created: 2024-05-28
## Parquet file structure
![[Pasted image 20240921091933.png]]


## Page pruning
Page index (aka page metadata)

Footer of a parquet file contains metadata for row groups, with in each row group, maintained a so called:  
- ColumnIndex: for each ColumnChunk in a rowgroup, the basic statistics (min,max,row_count....) of each data page in the column chunk (notice that one column chunk correspond to mulitple data page of a column). This is useful to check if a Data page is worth scanning at all, if based on statistics, we know that there is no such value matching the scanning predicate (e.g based on min..max)
- OffsetIndex: Mostly to quickly navigate to the first position of a given data page. This is because, there are chances that some data page will be skip (e.g data page 1, 2 or 3), thus the first data page that need to be scanned is data page 4, and the information of what is the offset to the first row to this data page 4 need to be stored somewhere, and hence this structure

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

