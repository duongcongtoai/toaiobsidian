# Recursive unnest
Created: 2024-06-03

### List -> field access -> List
```

select * from unnest(unnest(
    [struct([1,2,3]),struct([4,5,6])]
)['c0']);
```
#### Logical plan:
unnest(arr_field)
projection: structcol.field_access(fieldname) as arr_field
unnest(array_of_struct) -> structcol
projection: column1
scan table

### List -> struct
```
select * from unnest(unnest(
    [struct([1,2,3],[4,5,6]),struct([7,8,9],[10,11,12])]
));
```
#### Logical plan
unnest(struct_col)
unnest(struct_arr) -> struct col
Scan table
### List -> List
```
select * from unnest(unnest(
    [[1,2,3],[4,5,6]]
));
```
#### Logical plan
unnest(list_int) -> int
unnest(list_lint_int) -> list_int
scan table

### Expr tree as followed

unnest
	unnest
		2d_arr	
		
Trans into
```
project(item), other_col
	unnest(1d_arr) as item
		unnest(2d_arr) into 1d_arr
			projection(2d_arr), other col
```


unnest
	unnest
		struct_col
Trans into
```
project(multiple_col.col), other_col
	unnest(struct_col) as multiple cols
		unnest(struct_arr) into struct_col
			projection(struct_arr), other col
```


unnest
	udf::field_access c0
		unnest
			struct_arr_col
			
```

project()
unnest(co_field) as unnest_col_field, other_col
project(get_field(struct_col),'c0') as co_field, other_col
unnest(struct_arr) into struct_col
projection(struct_arr), other col
```
			

Given a list of exprs: a,b,c where a is an expr containing unnest children expr

unnest
udf::field_access
unnest(a_inner)
inner_projection: a_inner, b,c

	in order to do that, tree transformation as followed

#### step1:

unnest
	udff::field_access(alias of unnest(a_inner))
outer_proj: a_inner_alias, b, c
unnest(a_inner) as alias
inner_proj: a_inner, b, c

#### step2:



projection: structcol.field_access(fieldname) as arr_field

### Debugging
```
transforming expr: Unnest(Unnest { expr: ScalarFunction(ScalarFunction { func: ScalarUDF { inner: GetFieldFunc { signature: Signature { type_signature: Any(2), volatility: Immutable } } }, args: [Unnest(Unnest { expr: Column(Column { relation: None, name: "column1" }) }), Literal(Utf8("c0"))] }) })
```

## DuckDB Recursive unnest
Note: unnest has a big difference in behavior between Postgres and D
Take this example
1.Postgres
```ignored
ate table temp (
 i integer[][][], j integer[]

ert into temp values ('{{{1,2},{3,4}},{{5,6},{7,8}}}', '{1,2}');
ect unnest(i), unnest(j) from temp;
```

Result
    1	1
    2	2
    3	
    4	
    5	
    6	
    7	
    8	
2. DuckDB
```ignore
    create table temp (i integer[][][], j integer[]);
    insert into temp values ([[[1,2],[3,4]],[[5,6],[7,8]]], [1,2]);
    select unnest(i,recursive:=true), unnest(j,recursive:=true) from
```
Result:

    ┌────────────────────────────────────────────────┬──────────────
    │ unnest(i, "recursive" := CAST('t' AS BOOLEAN)) │ unnest(j, "re
    │                     int32                      │              
    ├────────────────────────────────────────────────┼──────────────
    │                                              1 │              
    │                                              2 │              
    │                                              3 │              
    │                                              4 │              
    │                                              5 │              
    │                                              6 │              
    │                                              7 │              
    │                                              8 │              
    └────────────────────────────────────────────────┴──────────────
The following implementation refer to DuckDB's implementation


### The steps as followed

Given the operator definition as followed

given columsn: col1 `integer[][][]`, col2 `integer[]`

And unnest operation as followed
select unnest(col1,depth=3), unnest(col1,depth=1), unnest(col2, depth =1)
Will be equivalent to

unnest(depth_2_col1), unnest(col1), unnest(col2) from (
	select unnest(dept_1_col1),col1,col2 as depth_2_col1 from (
		select unnest(col1),col1,col2 as depth_1_col2
	)
)

Note that unnest on depth1 is performed at the outer most select, which means the inner column must be projected through out the inner subqueries

Thus the algorithm is as followed

Determine the highest unnest level

current depth = highest unnest level
For current depth > 0:
	for each (col_i , depth) in columns_to_unnest
		if depth == current depth 
			`temp[(depth,col)` = unnest(col_i)
		if depth > current_depth
			previous = `temp[(depth,col)]`
			`temp[(depth,col)]` = unnest(previous)
		

Given a batch of 
```
-----------------------------
|col1         | col2|
|integer[][][]| integer[]|
----------------------------
_            | _ 
_            | _ 
_            | _ 
-----------------------------
```
First loop will result

into a batch with the same schema, but with more rows (because of the copy during unnest)
and a temp batch as 
```
|depth=3,col1|
|integer[][] |
---------
_            |
_            |
_            |
-------
```

Second loop
```
| depth=3,col1|
| integer[]   |
--------
_             |
_             |
_             |
--------------
```

Third loop
```
| depth=3,col1| depth=1,col1 | depth=1,col2  |
| integer     | integer[][]  | integer       |
---------------------------------------------
_             |_             |_              |
_             |_             |_              |
_             |_             |_              |
_             |_             |_              |
```
Now combine the transformed batch with the schema
for index, col in batch
	if unnest belongs to a column to be unnested
		replace col with (unnested_cols)









```
Aggregate: groupBy=[[unnest_placeholder(make_array(Int64(1),Int64(2),Int64(3)),depth=1)]], aggr=[[]]
  Unnest: lists[unnest_placeholder(make_array(Int64(1),Int64(2),Int64(3)))|depth=1] structs[]
    Projection: make_array(Int64(1), Int64(2), Int64(3)) AS unnest_placeholder(make_array(Int64(1),Int64(2),Int64(3)))
      EmptyRelation

from this input select
singular unnest [Alias(Alias { expr: Column(Column { relation: None, name: "UNNEST(make_array(Int64(1),Int64(2),Int64(3)))" }), relation: None, name: "c1" })]
```
Prev
```
input Aggregate: groupBy=[[UNNEST(UNNEST(UNNEST(make_array(make_array(make_array(Int64(1)))))))]], aggr=[[]]
  Unnest: lists[UNNEST(UNNEST(UNNEST(make_array(make_array(make_array(Int64(1)))))))] structs[]
    Projection: UNNEST(UNNEST(make_array(make_array(make_array(Int64(1)))))) AS UNNEST(UNNEST(UNNEST(make_array(make_array(make_array(Int64(1)))))))
      Unnest: lists[UNNEST(UNNEST(make_array(make_array(make_array(Int64(1))))))] structs[]
```
```
select exprs [Alias(Alias { expr: Column(Column { relation: None, name: "UNNEST(UNNEST(UNNEST(make_array(make_array(make_array(Int64(1)))))))" }), relation: None, name: "c1" })]
```

More complex case
```
statement ok
CREATE TABLE temp
AS VALUES
    ([1,2,3],1,[9,10]),
    ([1,2,3],2,[11,12]),
    ([4,5,6],2,[13,14])


query TT
explain select unnest(column1), unnest(column3) c3 from temp group by c3, column1;
----
```

TODO:
make try_process_unnest_group only transform for interested unnest expr, not all of them

## References
1. 
tags: 