# Ownership & Borrowing

Ownership is the mechanism Rust uses to manage memory without a garbage collector. Every value
has exactly one owner at any point in time, and when the owner goes out of scope the value is
dropped. Borrowing lets you hand out temporary references — shared (`&T`) or exclusive (`&mut T`)
— without transferring ownership.

The articles in this section build from the ground up: move semantics first, then the borrow
checker's rules for references.
