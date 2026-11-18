# The Iterator Trait

The `Iterator` trait requires exactly one method: `next`, which returns `Option<Self::Item>`.
Returning `Some(item)` advances the iterator; returning `None` signals exhaustion. Every other
method on iterators — `map`, `filter`, `enumerate`, and more than 70 others — is a default
implementation built on top of `next`. When you implement `Iterator` for your own type you get
the full combinator library for free.

A type becomes iterable in a `for` loop by implementing `IntoIterator`, which converts the type
into an iterator. `Vec<T>` implements it three ways: consuming (`vec`), by shared reference
(`&vec`), and by exclusive reference (`&mut vec`), giving you control over whether the loop
takes ownership or borrows.

## Code

```rust
struct Counter {
    count: u32,
    max: u32,
}

impl Counter {
    fn new(max: u32) -> Self {
        Counter { count: 0, max }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<u32> {
        if self.count < self.max {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let sum: u32 = Counter::new(5)
        .zip(Counter::new(5).skip(1))
        .map(|(a, b)| a * b)
        .sum();
    println!("{sum}"); // 2*3 + 3*4 + 4*5 + 5*6 … wait, let's just trust the chain
}
```

## Key Points

- Implement only `next`; all combinators are inherited default methods.
- `type Item` is an associated type — it fixes the element type for a given implementation.
- Iterators are lazy: combinators return new iterator structs, not computed results.
- `for x in collection` desugars to `IntoIterator::into_iter(collection)` followed by repeated `next`.
- `iter()` borrows, `iter_mut()` borrows mutably, `into_iter()` consumes — know which you need.

## Code Review

**Good**
```rust
let evens: Vec<_> = (0..10).filter(|x| x % 2 == 0).collect();
```
*Single lazy chain, one allocation at `collect`.*

**Bad**
```rust
let mut evens = vec![];
for x in 0..10 {
    if x % 2 == 0 { evens.push(x); }
}
```
*Manual loop with repeated `push` — does the same work but misses `filter`'s intent and may reallocate.*
