### datafusion_physical_expr::analyze
given physical expr (can be a composite of other expr such as)

This function is called inside FilterExec ExecutionPlan trait, thus the expr is the predicate it self, like:
```
&predicate = BinaryExpr {
    left: Column {
        name: "id",
        index: 0,
    },
    op: Gt,
    right: Literal {
        value: Int64(3),
    },
}
```
or a composite predicate
```
 BinaryExpr {
    left: BinaryExpr {
        left: Column {
            name: "id",
            index: 0,
        },
        op: Gt,
        right: Literal {
            value: Int64(3),
        },
    },
    op: And,
    right: BinaryExpr {
        left: Column {
            name: "name",
            index: 1,
        },
        op: Lt,
        right: Literal {
            value: Utf8("toai"),
        },
    },
}
```

```
&predicate = BinaryExpr {
    left: BinaryExpr {
        left: BinaryExpr {
            left: Column {
                name: "age",
                index: 2,
            },
            op: Gt,
            right: Literal {
                value: Int64(10),
            },
        },
        op: And,
        right: BinaryExpr {
            left: Column {
                name: "id",
                index: 0,
            },
            op: Gt,
            right: Literal {
                value: Int64(3),
            },
        },
    },
    op: And,
    right: BinaryExpr {
        left: Column {
            name: "name",
            index: 1,
        },
        op: Lt,
        right: Literal {
            value: Utf8("toai"),
        },
    },
}
```

### Analyze function:
    let mut graph = ExprIntervalGraph::try_new(expr.clone(), schema)?;
build_dag

from a hiercachy of physical expr, build a graph with each node a wrapper of expr and its interval. If the expr is constant, bound is its value, else the initial bound is unbounded
```
    match graph
        .update_ranges(&mut target_indices_and_boundaries, Interval::CERTAINLY_TRUE)?
    {
        PropagationResult::Success => {
            shrink_boundaries(graph, target_boundaries, target_expr_and_indices)
        }
        PropagationResult::Infeasible => {
            Ok(AnalysisContext::new(target_boundaries).with_selectivity(0.0))
        }
        PropagationResult::CannotPropagate => {
            Ok(AnalysisContext::new(target_boundaries).with_selectivity(1.0))
        }
    }

```
target_indices_and_boundaries are vector of tuples(index in graph, known interval)
Known interval can be some previously inferred bounds from statistics

### update_range
given_range is initially \[true,true\]


TODO: review this function
find a way to trigger this function from cli for debug (PS: using a test called test_custom_filter_selectivity)
This function does not call analyze because the predicate/schema does not support stat estimation
