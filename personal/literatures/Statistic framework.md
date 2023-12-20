get through an example
select \* from test where value > 1;
expr is value > 1
boundaries are columns boundaries (value)

graph has root node equivalent to the expr and child are inferred expr (value,1) with their own bound
columns are collected
```
    let target_boundaries = context.boundaries;

    let a = format!("{:?}",expr);
    dbg!(a);
    let mut graph = ExprIntervalGraph::try_new(expr.clone())?;

    let columns: Vec<Arc<dyn PhysicalExpr>> = collect_columns(expr)
        .into_iter()
        .map(|c| Arc::new(c) as Arc<dyn PhysicalExpr>)
        .collect();

    dbg!(&graph);
    let target_expr_and_indices: Vec<(Arc<dyn PhysicalExpr>, usize)> =
        graph.gather_node_indices(columns.as_slice());

    let mut target_indices_and_boundaries: Vec<(usize, Interval)> =
        target_expr_and_indices
            .iter()
            .filter_map(|(expr, i)| {
                target_boundaries.iter().find_map(|bound| {
                    expr.as_any()
                        .downcast_ref::<Column>()
                        .filter(|expr_column| bound.column.eq(*expr_column))
                        .map(|_| (*i, bound.interval.clone()))
                })
            })
            .collect();
    Ok(
        match graph.update_ranges(&mut target_indices_and_boundaries)? {
            PropagationResult::Success => shrink_boundaries(
                expr,
                graph,
                target_boundaries,
                target_expr_and_indices,
            )?,
            PropagationResult::Infeasible => {
                AnalysisContext::new(target_boundaries).with_selectivity(0.0)
            }
            PropagationResult::CannotPropagate => {
                AnalysisContext::new(target_boundaries).with_selectivity(1.0)
            }
        },
    )

```