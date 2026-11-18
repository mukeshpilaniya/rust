# Move Semantics

In Rust, assigning a heap-allocated value to another variable does not copy it — it *moves* it.
The original variable becomes invalid the moment ownership transfers, and the compiler enforces
this at compile time, not at runtime. This design eliminates an entire class of bugs — use-after-free,
double-free — without any runtime cost. Types that hold no heap resources (integers, booleans,
raw pointers) implement the `Copy` trait instead and are duplicated cheaply on assignment.

Understanding moves is the key to reading Rust error messages. When the compiler says
"value used after move," it means ownership left the current scope on a previous line and the
binding is now empty.

## Code

```rust
fn print_name(name: String) {
    println!("{name}");
} // `name` is dropped here

fn main() {
    let s = String::from("Mukesh");
    print_name(s);          // ownership moves into the function
    // println!("{s}");     // compile error: value used after move
}
```

## Key Points

- Assignment of a non-`Copy` type transfers ownership; the source binding is gone.
- Passing a value to a function is an assignment — ownership moves into the callee.
- Returning a value from a function moves it back to the caller.
- `Copy` types (`i32`, `bool`, `char`, `&T`, …) are silently duplicated on assignment.
- `Clone` is an explicit, potentially expensive deep copy; `Copy` is implicit and always cheap.

## Code Review

**Good**
```rust
fn greet(name: &str) {
    println!("Hello, {name}");
}
let s = String::from("Mukesh");
greet(&s);          // borrow — caller keeps ownership
greet(&s);          // can call again
```
*Borrowing instead of moving lets the caller reuse `s` without cloning.*

**Bad**
```rust
fn greet(name: String) {
    println!("Hello, {name}");
}
let s = String::from("Mukesh");
greet(s.clone());   // needless clone just to keep `s` alive
greet(s.clone());
```
*Taking `String` by value forces a heap allocation per call; `&str` costs nothing extra.*
