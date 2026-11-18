# Trait Objects

A trait object — written `dyn Trait` — is a fat pointer that pairs a data pointer with a vtable
at runtime. It enables *dynamic dispatch*: the method to call is looked up in the vtable at
runtime rather than resolved at compile time. This means you can store values of different
concrete types behind a single `Box<dyn Trait>` or `&dyn Trait`, which is impossible with
generics (where the type must be known at compile time).

The cost of dynamic dispatch is a pointer indirection per method call and the loss of
monomorphisation optimisations. For most use cases the cost is negligible, but it matters in
tight inner loops. Trait objects also impose an *object safety* requirement: the trait cannot
have methods that return `Self` or use generic type parameters, because the vtable would then
need to encode an unbounded set of implementations.

## Code

```rust
trait Draw {
    fn draw(&self);
}

struct Circle;
struct Square;

impl Draw for Circle {
    fn draw(&self) { println!("drawing circle"); }
}

impl Draw for Square {
    fn draw(&self) { println!("drawing square"); }
}

fn render_all(shapes: &[Box<dyn Draw>]) {
    for shape in shapes {
        shape.draw();   // vtable lookup at runtime
    }
}

fn main() {
    let shapes: Vec<Box<dyn Draw>> = vec![
        Box::new(Circle),
        Box::new(Square),
    ];
    render_all(&shapes);
}
```

## Key Points

- `dyn Trait` is a fat pointer: 16 bytes (data pointer + vtable pointer) on 64-bit platforms.
- Use trait objects when the set of concrete types is not known at compile time.
- A trait is object-safe if no method uses `Self` by value or has generic type parameters.
- `Box<dyn Trait>` owns the value; `&dyn Trait` borrows it.
- `dyn Trait + Send + Sync` adds auto-trait bounds for use across threads.

## Code Review

**Good**
```rust
fn notify(sender: &dyn Notify) {
    sender.send("hello");
}
```
*`&dyn Notify` avoids boxing when ownership isn't needed — cheaper and more flexible.*

**Bad**
```rust
fn notify(sender: Box<dyn Notify>) {
    sender.send("hello");
}
```
*`Box` forces a heap allocation at the call site even though the function only needs to borrow.*
