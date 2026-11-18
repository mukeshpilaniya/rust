# Traits & Generics

A trait is Rust's mechanism for defining shared behaviour. It is similar to an interface in other
languages, but stricter: you cannot add methods to a trait after the fact, and the compiler
resolves which implementation to call at compile time by default (static dispatch). Generics let
you write functions and types that work over any type satisfying a trait bound, with the compiler
generating specialised code for each concrete type used.

The articles here cover the two dispatch strategies — static via generics and dynamic via trait
objects — and the mechanics of writing generic code with bounds.
