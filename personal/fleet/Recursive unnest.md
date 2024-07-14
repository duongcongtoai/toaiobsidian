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
	

## References
1. 
tags: 