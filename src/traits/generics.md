# Generics

Generics let you write a function or type once and have the compiler specialise it for every
concrete type it is called with. This is *monomorphisation*: at compile time, Rust generates a
separate copy of the code for each distinct type argument. The result is code that runs as fast
as if you had written a dedicated version by hand, with zero runtime overhead — unlike dynamic
dispatch via trait objects.

Trait bounds (`T: Display`, `T: Clone + Debug`) constrain the generic parameter so the compiler
knows which methods are available inside the function. The `where` clause is an alternative
syntax for the same bounds, preferred when the bound list grows long enough to obscure the
function signature. `impl Trait` in an argument position is syntactic sugar for a generic
parameter with a single bound; in a return position it means "some concrete type implementing
this trait" (useful for returning closures or iterators without naming the type).

## Code

```rust
use std::fmt::Display;

fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn print_pair<T, U>(a: T, b: U)
where
    T: Display,
    U: Display,
{
    println!("{a} and {b}");
}

fn make_adder(x: i32) -> impl Fn(i32) -> i32 {
    move |y| x + y
}
```

## Key Points

- Monomorphisation generates one concrete function per type argument — no runtime overhead.
- Bounds written as `T: TraitA + TraitB` require the type to implement both traits.
- `where` clauses improve readability when multiple type parameters each have several bounds.
- `impl Trait` in argument position is sugar for an anonymous generic; in return position it hides the concrete type.
- Blanket implementations (`impl<T: Display> MyTrait for T`) apply a trait to any type satisfying a bound.

## Code Review

**Good**
```rust
fn print<T: Display>(value: &T) {
    println!("{value}");
}
```
*Generic over `Display` — works with any printable type, no boxing, zero overhead.*

**Bad**
```rust
fn print(value: &dyn Display) {
    println!("{value}");
}
```
*`&dyn Display` adds a vtable indirection for a function that is always called with a known type — use generics instead.*
