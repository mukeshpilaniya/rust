# Lifetime Annotations

A lifetime annotation does not change how long a value lives — it tells the compiler the
*relationship* between the lifetimes of multiple references. The syntax `'a` is just a label;
the name carries no inherent meaning. When a function takes two references and returns one,
the compiler needs to know which input the output borrows from, because that determines how
long the returned reference may be used.

Lifetime parameters are declared in angle brackets after the function name, alongside type
parameters. Structs that hold references also require lifetime annotations so the compiler
can verify that the struct never outlives the data it borrows.

## Code

```rust
// The returned reference lives at least as long as both inputs.
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() >= y.len() { x } else { y }
}

struct Excerpt<'a> {
    text: &'a str,   // struct cannot outlive the string it borrows
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first = novel.split('.').next().unwrap();
    let excerpt = Excerpt { text: first };  // `excerpt` tied to `novel`'s lifetime
    println!("{}", excerpt.text);
}
```

## Key Points

- `'a` on a function means "these references share a lifetime region" — the output lives no longer than the shorter input.
- Lifetime annotations are checked at the call site, not inside the function body.
- `'static` means the reference is valid for the entire program — string literals are `'static`.
- Structs holding references must annotate every reference field.
- Generic lifetime bounds (`T: 'a`) mean "T must outlive 'a", not "T has lifetime 'a".

## Code Review

**Good**
```rust
fn first_word(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}
```
*Elision applies: the output borrows from the single input, no annotation needed.*

**Bad**
```rust
fn first_word<'a, 'b>(s: &'a str, _ctx: &'b str) -> &'a str {
    s.split_whitespace().next().unwrap_or("")
}
```
*`'b` is declared but unused; `_ctx` is unrelated to the output — misleading and noisy.*
