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


## References
1. https://www.youtube.com/watch?v=NVKujPxwSBA 
2. slide: https://docs.google.com/presentation/d/1D3GDVas-8y0sA4c8EOgdCvEjVND4s2E7I6zfs67Y4j8/edit#slide=id.p
3. How query engine works: https://howqueryengineswork.com/00-introduction.html
tags: 