# Decorrelate dependent join
Created: 2024-11-09

## Undestand correlated subqueries
![[Pasted image 20241125195948.png]]

Signature
Difference from cross join?
given this query

```
select * from a join b on a.id=b.id
```

The join condition only include certain values

While the correlated subquery looks like

```
	Select * from a where exists (
		select * from b where b.id=a.id
	)
```
Where every record of a there is a need to evaluate with every of the record of b

Free variable in the correlated subquery appear in expr
```
b.id=a.id
```
is "a.id"
![[Pasted image 20241125201151.png]]

To evaluate the expr T2 `all b that has b.id=a.id`, the free variable `a.id` must be an attribute produced by T1 "select * from a"

Let's take another example
## Group by
![[Pasted image 20241109112733.png]]

e is the base relation
A is the set of attributes within e, used for grouping (group by columns)
a: f(y) are the aggregation result of one or more f(y), where y represents the group all records that have the same set of attribute A

The flow is:
For a given set of group by keys A in the outer relation, like
select a,b, ..... group by (a,b)
then a,b is the grouping key set

Do projection on base relation on these groupking key set A
for each unique combination of value of A, find respective rows that share the same values in these attributes


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

From the dependent join of T1 and T2, this step introduce an extra step:
- D is the projection of all columns from T1 which T2 dependent on
- perform dependent join from D to T2 instead of T1 to T2
- Then join T1 with (D dependent join T2)

## Break down
Find out if in Datafusion already support simple unnest usecase
Done: already implemented

## Major point
![[Pasted image 20250112034345.png]]

## References
1. 
tags: 