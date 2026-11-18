# Combinators

Combinators are the default methods on `Iterator` that transform or consume a sequence without
requiring a manual loop. They fall into two groups: *adapters* return a new iterator (lazy,
no work done yet), and *consumers* drive the iterator to completion and produce a final value.
Adapters can be chained freely in any order; the work only happens when a consumer is called at
the end of the chain.

The most common adapters are `map` (transform each element), `filter` (keep matching elements),
`flat_map` (transform then flatten), `take` / `skip` (slice the sequence), and `enumerate`
(pair each element with its index). The most common consumers are `collect`, `sum`, `fold`,
`any`, `all`, and `for_each`. Knowing which is which matters because calling an adapter without
a consumer produces a warning: you built an iterator that nothing ever polls.

## Code

```rust
fn word_lengths(text: &str) -> Vec<(usize, &str)> {
    text.split_whitespace()
        .filter(|w| w.len() > 3)          // adapter: skip short words
        .enumerate()                       // adapter: attach index
        .map(|(i, w)| (i + 1, w))         // adapter: 1-based index
        .take(5)                           // adapter: cap at 5 results
        .collect()                         // consumer: materialise into Vec
}

fn main() {
    let results = word_lengths("the quick brown fox jumps over the lazy dog");
    for (n, word) in &results {
        println!("{n}: {word}");
    }
}
```

## Key Points

- Adapters are lazy — no iteration happens until a consumer is reached.
- `fold(init, |acc, x| ...)` is the generalisation that most other consumers are built on.
- `flat_map` is `map` + `flatten`; use it when each element produces a sequence.
- `chain` concatenates two iterators without allocating a new collection.
- `peekable()` wraps an iterator so you can inspect the next element without consuming it.

## Code Review

**Good**
```rust
let total: i32 = scores.iter().filter(|&&s| s > 0).sum();
```
*One lazy pass, no temporary collection.*

**Bad**
```rust
let positive: Vec<_> = scores.iter().filter(|&&s| s > 0).collect();
let total: i32 = positive.iter().sum();
```
*Allocates `positive` only to immediately sum it — split into two passes for no reason.*
