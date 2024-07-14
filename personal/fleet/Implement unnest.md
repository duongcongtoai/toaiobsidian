## Usecase

unnest(column) used inside predicate
```
select * from adlog where unnest(sub_request)['id'] = '1';
type_coercion
caused by
This feature is not implemented: Unnest should be rewritten to LogicalPlan::Unnest before type coercion
```

support unnest in subquery
```
            select unnest(sub_request) as sub_req, unnest(auctioning)['competitors'] as competitors from adlog 
            where exists (
                select * from adlog where unnest(sub_request)['id'] = '1';
            )
```

support multiple unnest in from clause
```
D select a1,b1 from a, unnest(a.a) as a1, unnest(a.b) as b1;
┌───────────────────┬───────────────────┐
│        a1         │        b1         │
│ struct(a integer) │ struct(b integer) │
├───────────────────┼───────────────────┤
│ {'a': 2}          │ {'b': 4}          │
│ {'a': 2}          │ {'b': 5}          │
│ {'a': 2}          │ {'b': 6}          │
│ {'a': 9}          │ {'b': 10}         │
│ {'a': 9}          │ {'b': 10}         │
│ {'a': 1}          │ {'b': 4}          │
│ {'a': 1}          │ {'b': 5}          │
│ {'a': 1}          │ {'b': 6}          │
│ {'a': 3}          │ {'b': 4}          │
│ {'a': 3}          │ {'b': 5}          │
│ {'a': 3}          │ {'b': 6}          │
├───────────────────┴───────────────────┤
│ 11 rows                     2 columns │

```
May have to rewrite unnest in plan_selection

Revisit: https://github.com/apache/datafusion/pull/9069/files



[[Recursive unnest]]