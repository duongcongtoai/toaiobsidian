## Usecase
```
select unnest(struct_col) from sometable
```

```
select struct_col.subfield from sometable
```

```
select struct from sometable, unnest(struct_arr) as struct
```

Current problem
Rewriting
```
rewriting plan Unnest: 0
  Projection: unnest_table.column1 AS unnest(unnest_table.column1)
    TableScan: unnest_table
---
Give its new expr [Column(Column { relation: None, name: "name0" })]
---
and new input [Projection: unnest_table.column1 AS unnest(unnest_table.column1)
  TableScan: unnest_table]
==========
External error: query failed: DataFusion error: type_coercion
caused by
Schema error: No field named name0. Valid fields are "unnest(unnest_table.column1)".
[SQL] select unnest(column1) from unnest_table;
at test_files/unnest_debug.slt:29

```
new expr is the result after calling unnest node to iterate through each of its expression and apply some rewrite function in type_coersion node
Some how new expr returns the unnested column and passed down to the input, which does not have this column in the schema
```
rewriting plan Projection: unnest_table.column1 AS unnest(unnest_table.column1)
  TableScan: unnest_table
---
Give its new expr [Alias(Alias { expr: Column(Column { relation: Some(Bare { table: "unnest_table" }), name: "column1" }), relation: None, name: "unnest(unnest_table.column1)" })]
```
maybe at the time projection is rewritten, also detect if a unnest is an expr inside, consider to give projection new input of unnest


Current error found
        select subreq, unnest(auc) from (
            select unnest(sub_request) as subreq, unnest(auctioning)['competitors'] as auc from adlog 
            limit 1
        )

Arrow error: Invalid argument error: all columns in a record batch must have the same length
