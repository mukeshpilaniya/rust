# Lifetime Elision

Lifetime elision is a set of three inference rules that let the compiler fill in lifetime
annotations automatically when the relationship is unambiguous. These rules were added to Rust
specifically because the vast majority of real functions either take a single reference and
return one derived from it, or take `&self` and return something tied to self. Writing `'a`
everywhere in those cases would be mechanical noise with no information value.

The three rules apply in order. First, every reference parameter gets its own distinct lifetime.
Second, if there is exactly one input lifetime, it is assigned to all output lifetimes. Third,
if one of the input parameters is `&self` or `&mut self`, the lifetime of `self` is assigned to
all output lifetimes. If none of these rules fully determine the output lifetime, the compiler
requires an explicit annotation.

## Code

```rust
// All three functions are identical after elision is applied.

// Written with explicit annotations:
fn get_ref<'a>(x: &'a i32) -> &'a i32 { x }

// Written with elision (rule 2 applies — one input, one output):
fn get_ref(x: &i32) -> &i32 { x }

struct Cache {
    value: String,
}

impl Cache {
    // Rule 3: output borrows from &self
    fn value(&self) -> &str {
        &self.value
    }
}
```

## Key Points

- Rule 1: each `&` parameter gets its own lifetime label (e.g., `'a`, `'b`).
- Rule 2: if exactly one input reference exists, its lifetime flows to every output reference.
- Rule 3: if `&self` or `&mut self` is present, its lifetime flows to every output reference.
- If the rules produce a unique answer the annotation is elided; otherwise you must write it.
- Elision is purely a syntactic shortcut — the compiler still checks the full lifetime graph.

## Code Review

**Good**
```rust
impl Config {
    fn name(&self) -> &str { &self.name }
}
```
*Rule 3 applies cleanly — no annotation needed, intent is clear.*

**Bad**
```rust
impl Config {
    fn name<'a>(&'a self) -> &'a str { &self.name }
}
```
*Explicit `'a` here is redundant; elision already handles it and the annotation adds visual clutter.*
