# Decorrelate dependent join
Created: 2024-11-09
## Group by
![[Pasted image 20241109112733.png]]

e is the base relation
A is the set of attributes within e, used for grouping (group by columns)
a: f(y) are the aggregation result of one or more f(y), where y represents the group all records that have the same set of attribute A

## Simple unnesting
Given a query 
```
select * from lineitem l1 where exists (
		select * from lineitem l2 where l1.orderkey=l2.orderkey
)
```

in Datafusion this can be expressed as
```
select{
	predicate: Vec<Expr>,
	...
}
```
and predicate as the form of
```
[
	Exist(Subquery{
		Query: Select{
			from: [inner_relation]	
			predicate: Vec[
				{
					outerColumn.column2
					eq
					inner_relation.column1
				}
			]
		}
	})
]
```

This can be transformed into
```
select{
	predicate: vec[]
	from: outer_rela,inner_rela join using column1,column2
}
```
### General unnesting
![[Pasted image 20241110114003.png]]

Reference from paper
first step: transform the dependent join into a "nicer" version of dependent join



## References
1. 
tags: 