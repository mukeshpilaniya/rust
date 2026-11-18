# Lifetimes

A lifetime is the region of code for which a reference is valid. Rust tracks lifetimes at compile
time to prove that no reference can outlive the data it points to. In the majority of functions
the compiler infers lifetimes automatically through a set of elision rules, so you only write
explicit annotations when the compiler cannot figure out the relationship on its own.

The articles here start with explicit annotation syntax and then cover the elision rules that
let you omit annotations in the common cases.
