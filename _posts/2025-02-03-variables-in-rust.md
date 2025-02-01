---
title: Rust Syntax Basic: Variables, Constants and Shadowing
date: 2024-02-03 10:00:00
categories: [Rust]
tags: [Rust, 90 Days of Rust]
---

As we have already installed Rust on local, lets explore
basics of Rust syntax, focusing on variables, constants and
shadowing.

## Variables

In Rust, variables are immutable by default, which means once
value is assigned to the variable then it cannot be changed.
To make variable mutable, `mut` keyword must be used.

**Example:**

```rust
fn main() {
    let x = 5;          // Immutable variable
    let mut y = x * 2;  // Mutable variable

    println!("x: {}; y: {}", x, y);

    y = 20;

    println!("Updated Values => x: {x}; y: {y}");
}

```

**Key Points:**

- Use `let` to declare variables.
- Variables are immutable by default; Use `mut` for mutability.

## Constants

Constants are always immutable and must be explicitly typed.
They are declared using the `const` keyword and are valid for the entire program lifetime.

**Example:**

```rust
fn main() {
    const PI: f32 = 3.14159;  // Constant with explicit type
    println!("The value of PI is: {}", PI);
}
```

**Key Points:**

- Use `const` for declaring constants and constants must declare type annotations.
- Cannot use `mut` with constants.

## Shadowing

Shadowing allows you to reuse a variable name by creating a new variable with the same name.
This is different from mutability because the original variable is effectively "shadowed" by the new one.

**Example:**

```rust
fn main() {
    let x = 5;
    let x = x + 1; // Shadowing creates new variable `x`

    {
        let x = x * 2; // Shadowing within a scope
        println!("Inner x: {}", x); // Output: 12
    }
    println!("Outer x: {}", x); // Output: 6
}
```

**Key Points:**

- Shadowing creates a new variable under the same name.
- Useful for transforming data without mutating the original variable.
- Works across different scopes.

### Summary

- **Variables**: Immutable by default; use `mut` for mutability.
- **Constants**: Always immutable, require explicit typing, and use `const` to declare.
- **Shadowing**: Reuse variable names to create new variables, even in nested scopes.

These concepts form the foundation of Rust’s syntax and help ensure safe and predictable code.
Start experimenting with these basics, and you’ll quickly get comfortable with Rust’s unique approach! 🦀
