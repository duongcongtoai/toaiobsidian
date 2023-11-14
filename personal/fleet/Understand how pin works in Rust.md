# Understand how pin works in Rust
Created: 2023-10-29

Given this type:
```
pub type BoxStream<'a, T> = Pin<alloc::boxed::Box<dyn Stream<Item = T> + Send + 'a>>;
```
This is a type returned by a scan operator (a physical operator in the context of datafusion), the output of which can be 

Pinning is an annotation in Rust that allow developer to say to compiler that "hey, ensure my type will never be moved, even if i write the code"

For example i have a variable that reference some field of other type.
Saying a -> b.a
b then being moved somewhere else, now 

PhantomPinned implement !Unpin
```
impl !Unpin for PhantomPinned {}
```
Every type that has a field implement !UnPin also impl !Unpin

impl !Unpin means it cannot be unpin, meaning once the type is pinned, it cannot be moved.

In Rust, by saying you pinning something, compiler may or may not actually pin the memory of the object. 
## Why pin
Ensure object remains in a fixed location in memory
- Pinning ensures that certain types of data are always available in memory, even if the computer swaps out other parts of the program.

For example in the future, every impl of future trait must 
```
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
```
Pin<&mut self>?
because a future can be polled by multiple threads. For example
- t1.poll(future)
-  park
- next time the future is ready to be polled again, t2.poll(future)
Now, the thing special about future is that it usually has self referential fields, meaning:
```
some_future{
	a: integer,
	b: pointer_to_a
}
```
on passing execution from t1 to t2, the value of this future will be moved, meaning:
```
moved_some_future {
	a: integer
	b: pointer_to_old_future_a
}
```
This is invalid because b now points to a moved location
Pin syntax allow these types to be safe from this self-referential nature of themselves
### When is a future start to be pinned?
from the time it get polled for the first time, or by the time it is created (by its self-referential nature)
### What is pin project?

## References
1.  https://blog.cloudflare.com/pin-and-unpin-in-rust/
tags: #rust #pin #future
