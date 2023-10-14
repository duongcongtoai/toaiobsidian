## Brief of vectorization
Each dbms maintain a precompiled version of primitives executor (for example str_cols_compare function, int_cols_compare function) that operates on a set of tuple. The incentive is to amortize dbms's interpretation decision???
TODO: what does interpretation decision here, how much does it cost in the performance
## What is query compilation and why it is efficient
Given an example from this paper:

Everything You Always Wanted to Know About Compiled and Vectorized Queries But Were Afraid to Ask https://15721.courses.cs.cmu.edu/spring2023/papers/10-vect-vs-comp/p2209-kersten.pdf

For example given this query
```
select * where color="green" and tires =4
```

Then the 2 predicates are non-blocking to each other, the dbms can compile 2 predicates to be executed by one physical executor (a compiled version)

Integrated (data-centric compilation)
```
vec<int> sel_eq_row(vec<string> col, vec<int> tir)
{
    vec<int> res;
    for(int i=0; i<col.size(); i++) // for colors and tires
        if(col[i] == "green" && tir[i] == 4) // compare both
            res.append(i) // add to final result
    return res;
}
```

While with the vectorization approach, there are for loop with number of iteration for each loop is nearly doubled compared to the data-centric approach.
Decomposed (vectorized)

```
vec<int> sel_eq_string(vec<string> col, string o)
{
    vec<int> res;
    for(int i=0; i<col.size(); i++) // for colors
        if(col[i] == o) // compare color
            res.append(i) // remember position
    return res;
}

vec<int> sel_eq_int(vec<int> tir, int o, vec<int> s)
{
    vec<int> res;
    for(i : s) // for remembered position
        if(tir[i] == o) // compare tires
            res.append(i) // add to final result
    return res;
}
```


![[vectorwise vs hyper.png]]
In the picture, here, in TPCH query 1 and 18, even though hyper has lower IPC, the performance wise it is still better than vectorwise because the total instructions need to be executed is significantly less

However in Q3 with the join, hyper is executing with fewer instruction overall,, but the ipc is way less than its counter part that the overall performance is inferior to vectorwise. So what is the cause? => the branch miss is killing hyper's performance.
Which make sense because in the case of join, in vectorwise, the likely hood that the code is bounced back and get more tuple is lower comparing to single tuple execution of hyper, because there is less branching

#hyper  #vectorwise #querycompilationa #vectorization
