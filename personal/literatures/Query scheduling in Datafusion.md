# Query scheduling in Datafusion
Created: 2023-10-19

Note:
Looks like datafusion does not have sofisticted query schedulling

Datafusion is a query engine

## Usecases
- analytic db: influx iox, ceresdb,ballista
- reuse query language
- fileformat transcoding (json to parquet with filter, order)
## Extensibility
User defined function
User defined aggregate
User defined optimizer (logical and physical)
User defined Logical plan node
User defined physical plan node
User defined table provider
User defined catalog provider
## Fundamentals
### Record batch
- cols
- schema

Each record batch contains array of vector
Each vector is a sequence of column values
### Logical plan
is a dataflow graph from bottom to top
[[How window logical operator works and how to build it in Datafusion]]
## Data types
[[Datafusion datatypes]]

## Logical optimizor
### pushdown
Goal is to minimize bandwidth/data traversed from node to node
How PushdownLimit works
### simplify
evaluate the expr in the logical plan to see if any expr can be evaluated once rather than row to row
- SimplifyExpressions
- UnwrapCastInComparison
Or remove unused nodes
- Merge projection
-  EliminateFilter/Projection/CrossJoin/Limit/DuplicateExpr
Rewrite subquery to join
- DecorrelateWhereExists/DecorrelateWhereIn => DecorrelatePredicateSubquery
- ScalarSubqueryToJoin
### optimize join predicate
- ExtractEquijoinPredicate => RewriteDIsjunctivePredicate

ExtractEquiJoinPredicate has an insight that equijoin join predicate is important and has optimization opportunity, thus the engine must detect those predicates

RewriteDIsjunctivePredicate gives a scenario of tpch 19, where condition is a combination of predicates through "Or", each of the smaller predicate containing the same predicate (equijoin predicate), then those predicates are not discjuctive => RewriteDisjunctivePredicate extract the equijoin predicate shared by those 3 predicates, so that all the "or" predicates are disjuctive
EliminateOuterJoin: insight is that rewrite outer join to inner join anywhere that is possible, because inner join has more optimization opportunity than outer join. If a query has outer join, but later on there exist a predicate that returns false on the fact that "outer colums" returned null for non matching rows. For example 
```
select a.id,b.value from a left join b where b.some_col = something
```
then should there exist rows that exist id from a but having no matching rows from b, the predicate b.some_col always return false. thus we can safely ignore all the non-matching-rows-in-b at the beginning and this simply becomes inner join
### optimize distinct
- SingleDisctinctToGroupBy (by default every distinct can be converted to groupby)
## Expression Tree
Let's say i have an expression

```
path='/something'
```
In datafusion program, this is represented as a tree:
root: BinaryExpr{op: eq}
left: Column
right: Literal {type: ScalarValue:UTF8, value: '/something'}
## Task scheduling
A thread on datafusion slack:
tldr: Datafusion needs a custom scheduler instead of relying on tokio and rayon
[[Ballista Query Stage scheduler]]: https://github.com/apache/arrow-datafusion/issues/1704
## References
1. https://www.youtube.com/watch?v=NVKujPxwSBA 
2. slide: https://docs.google.com/presentation/d/1D3GDVas-8y0sA4c8EOgdCvEjVND4s2E7I6zfs67Y4j8/edit#slide=id.p
3. How query engine works: https://howqueryengineswork.com/00-introduction.html
4. https://the-asf.slack.com/archives/C01QUFS30TD/p1623602908302500?thread_ts=1623523766.291500&cid=C01QUFS30TD

tags: #queryscheduling #datafusion