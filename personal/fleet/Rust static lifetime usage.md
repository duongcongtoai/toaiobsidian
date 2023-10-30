# Rust static lifetime usage
Created: 2023-10-26

Hello, So, I was learning Tokio, and in Tokio runtime, it asks for `static future to be consumed.

So, i just want to know that when the 'static resources get freed and can we manually free those 'static resources once we are done with it ?
The `'static` lifetime implies that the object _can_ outlive any lifetime. It doesn't necessarily mean it only gets dropped when the program terminates. It just means Rust doesn't have to enforce that this object goes out of scope before any other object does.

For example, a `&'static str` has a `'static` lifetime, but so does an `i64`, or a `String`. This is why functions like

```
fn take<T : Clone + 'static>(t : T);
```

can have strings and integers as argument.

In the context of Tokio, which is essentially a concurrency framework, this makes sense: a `'static` resource doesn't have to be destroyed before any other resource, and it can live as long as needed. On the other hand, `'a` resources do have a lifetime and must be destroyed before their parent goes out of scope.

We are, however, working with concurrency. This means that we can never guarantee that something in one thread gets executed before something in another thread, so we cannot promise that an object in one thread gets destroyed before an object in another thread goes out of scope: it's always possible to park the first thread until the second thread is done computing, and the compiler itself is not smart enough to reason about locks.

Therefore, in order to send an object to another thread, requiring that it can live on its own is the safest option.

## References
1. 
tags: #rust #lifetime