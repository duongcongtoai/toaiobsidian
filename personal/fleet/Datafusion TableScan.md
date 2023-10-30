# Datafusion TableScan
Created: 2023-10-26

Convert table source to table provider
- projection: `Option<Vec<usize>>`
- filter: `Vec<Expr>`

```
    expr.transform(&|expr| {
        Ok({
            if let Expr::Column(c) = expr {
                let col = Column {
                    relation: None,
                    name: c.name,
                };
                Transformed::Yes(Expr::Column(col))
            } else {
                Transformed::No(expr)
            }
        })
    })
```

transform is a util to apply certain function to a node of a tree

table provider provide execution plan to scan the specified object

### ListingTable as TableProvider implementation
```
supports_filter_pushdown
```
This method is useful on optimizor
Why 


## References
1. 
tags: 