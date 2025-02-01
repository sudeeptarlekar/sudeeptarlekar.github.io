---
title: Installing Rust on local
date: 2025-02-02 10:00:00
categories: [Rust]
tags: [Rust, 90 Days of Rust]
---

## Introduction

As we saw, Rust is redefining the programming in
[previous](/posts/introduction-to-rust) blog post,
lets install Rust on local and write our first `Hello, World!` program.

## Install Rust

Use `rustup`, the official Rust installer, to set up Rust on your system.

### macOS/Linux

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Follow prompts and add Rust to your PATH:

```bash
source $HOME/.cargo/env
```

### Windows

Download `rustup-init.exe` from [rustup.rs](https://rustup.rs/#) and follow the installer.
Make sure to install the Visual Studio C++ Build Tools when prompted.

### Verify Installation

Check if Rust is installed properly.

```bash
rustc --version
```

## Meet Cargo

Cargo is Rust’s build system and package manager.
It comes bundled with Rust.
Verify its installation:

```bash
cargo --version
```

## Hello, World!

1. Create a new project
```bash
cargo new hello_world
```

2. `src/main.rs` already contains following code.
```rust
fn main() {
    println!("Hello, world!");
}
```

3. Build and rust hello world program:
```bash
cargo run
```

Output:

```bash
Hello, World!
```

## What’s Next?

- Explore the [Rust Book](https://doc.rust-lang.org/book/) for in-depth learning.
- Use `cargo build` to compile or `cargo check` to lint your code.
- Add dependencies in `Cargo.toml` and experiment with Rust’s ecosystem.
