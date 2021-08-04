+++
title = "Rust Ownership Learning Notes"
description = "Just some personal notes on this topic."
date = 2020-12-12
+++

Recently I decided to rewrite a small Python project in Rust -- just to
practice with a problem I had to solve a few months ago in my job. The goal is
to have a better notion of the language with situations I frequently find in
the projects I work on.

Among the various messages I got from the compiler, the majority of them were
related to ownership. So today I started learning more about it and list some
key points I found along the way.

Since this is an ongoing project and I will do my best keep it updated with any
interesting information I find while studying this language. I hope this can be
helpful for those who are in this same learning journey.

All the resources I used are listed at the end of this post. If you find any
error, please contact me so I can fix it as soon as possible.

# Learning Notes

> Keeping track of what parts of code are using what data on the heap,
> minimizing the amount of duplicate data on the heap, and cleaning up unused
> data on the heap are problems that ownership addresses.
>
> Rust Book

## Stack

- The stack stores values in the order it gets them and removes the values in
the opposite order.
- All data stored on the stack must have a known fixed size. Data with an
unknown size at compile time or a size that might change must be stored on the
heap.
- Because the pointer has a known, fixed size, you can store the pointer on the
stack, but when you want the actual data, you must follow the pointer.
- When your code calls a function, the values passed into the function
(including, potentially, pointer to data on the heap), and the function's local
variables get pushed onto the stack. When the function is over, those values
get popped off the stack.
- Pushing values onto the stack is not considered allocating. It is faster than
allocating on the heap, because the allocator never has to search for a place
to store new data. The location is always at the top of the stack.
- Data is copied (see *copy types* -- types that implement the `Copy trait`).
*Copy* is a cheap operation on stack.
- We can only access things in our stack frame.

## Heap

- The heap is less organized and allocating space on the heap requires more
work: when you put data on the heap, you request a certain amount of space. The
memory allocator finds an empty spot that is big enough, marks it as being in
use, and returns a *pointer*, which is the address of that location.
- Accessing data on the heap is slower than accessing data on the stack because
you have to follow a pointer to get there.
- Allocating a large amount of space on the heap can also take time.
- Data is moved/cloned. Clone is an expensive operation (see *deep copy*).
*Move* is a cheap operation (because it is dealing with a pointer).

## Ownership

> In Rust, whether a value is on the stack or the heap has an effect on how the
> language behaves and why you have to make certain decisions. This is how Rust
> avoids using a garbage collector. But once you understand ownership, you
> won't need to think about the stack and the heap very often.
>
> Rust Book

- Ownership is how Rust achieves its largest goal: memory safety.
- None of the ownership features slow down your program while it's running.
- Variables are in charge of freeing their own resources.
- Not all variables own resources (e.g. references).
- [Mutability of data can be changed when ownership is
transferred](https://doc.rust-lang.org/stable/rust-by-example/scope/move/mut.html).

### Benefits

- Runtime speed and no garbage collector.
- Better support for parallel and concurrent processing.
- Safety.

### Rules

1. Each value in Rust has a variable that's called its owner.
2. There can only be one owner at a time.
3. When the owner goes out of scope, the value will be dropped.

- Ownership begins with assignment and ens with scope.
- Reassignment moves ownership. Moving ownership makes the old variable invalid.
- Function arguments are similar to variable assignment and can take ownership.
The drop happens when the owned value drops out of scope.

```rust
fn function_that_takes_ownership(s: String) {
} // drop happens here
```

- It is possible to return the ownership, but we don't want to keep passing
around ownership. In this case, Rust offers the option to borrow.
- When functions have references as parameters instead of the actual values, we
won't need to return the values in order to give back ownership, because we
never had ownership

*TO BE CONTINUED...*

# References

- [Rust
Book](https://doc.rust-lang.org/stable/book/ch04-00-understanding-ownership.html)
- [Rust By Example](https://doc.rust-lang.org/stable/rust-by-example/)
- [Rust Ownership by
Example](https://depth-first.com/articles/2020/01/27/rust-ownership-by-example/)
- [JavaTpoint Rust Ownership](https://www.javatpoint.com/rust-ownership) has
nice illustrations about the use of the stack and the heap
- CoRecursive Podcast [Moves and Borrowing In
Rust](https://corecursive.com/016-moves-and-borrowing-in-rust-with-jim-blandy/)
- Video [Pointers and dynamic memory -- Stack vs.
Heap](https://www.youtube.com/watch?v=_8-ht2AKyH4)
([mycodeschool](https://www.youtube.com/user/mycodeschool))
- Video [Rust Ownership and
Borrowing](https://www.youtube.com/watch?v=lQ7XF-6HYGc) ([Doug
Milford](https://www.youtube.com/channel/UCmBgC0JN41HjyjAXfkdkp-Q))
- Video [Learning Rust: Memory, Ownership and
Borrowing](https://www.youtube.com/watch?v=8M0QfLUDaaA)
([YouCodeThings](https://www.youtube.com/channel/UC0yCXVwW6FdDQGYA-3OWXxw))
