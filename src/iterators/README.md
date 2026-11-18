# Iterators

Rust's iterator system is built on a single trait — `Iterator` — with one required method,
`next`. Everything else: `map`, `filter`, `fold`, `zip`, `chain`, and dozens more combinators,
are provided as default methods on that trait. Iterators are *lazy*: no work happens until a
consuming adaptor (`collect`, `for_each`, `sum`, …) or a `for` loop drives them. This means you
can compose a long chain of transformations with no intermediate allocations.

The articles here cover the trait itself and the most useful combinator patterns.
