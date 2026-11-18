# Borrowing

A borrow is a temporary reference to a value that does not transfer ownership. Rust enforces two
rules at compile time: you can have any number of shared (`&T`) references at the same time, or
exactly one exclusive (`&mut T`) reference — never both simultaneously. This rule is the borrow
checker's core invariant, and it guarantees that no two parts of a program can observe a mutation
at the same time, eliminating data races in safe Rust entirely.

The lifetime of a reference must not exceed the lifetime of the value it points to. The compiler
tracks this automatically in most cases via lifetime elision, and requires explicit annotations
only when the relationship is ambiguous.

## Code

```rust
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() >= b.len() { a } else { b }
}

fn main() {
    let mut v = vec![1, 2, 3];

    let first = &v[0];          // shared borrow
    println!("first = {first}");
    // v.push(4);               // compile error: cannot mutate while borrowed

    v.push(4);                  // shared borrow is no longer live here — OK
    println!("{v:?}");
}
```

## Key Points

- `&T` is a shared reference: many can coexist, none can mutate through it.
- `&mut T` is an exclusive reference: no other reference of any kind may exist simultaneously.
- References must always point to valid, live data — dangling references are a compile error.
- The borrow checker uses *Non-Lexical Lifetimes* (NLL): a borrow ends at its last use, not at the closing brace.
- Interior mutability (`Cell`, `RefCell`, `Mutex`) moves the exclusivity check to runtime when compile-time checking is too restrictive.

## Code Review

**Good**
```rust
fn sum(data: &[i32]) -> i32 {
    data.iter().sum()
}
```
*Takes a slice reference — works with `Vec`, arrays, and sub-slices without copying.*

**Bad**
```rust
fn sum(data: Vec<i32>) -> i32 {
    data.iter().sum()
}
```
*Consumes the `Vec` needlessly; the caller loses it after the call and can't pass a slice.*
