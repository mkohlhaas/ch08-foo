# Trait Objects Gain Methods via Blanket Impl

This example shows how a trait object (`dyn Trait`) can be given extra methods
by implementing another trait *for* the trait object itself:

```rust
impl<'a> B for dyn A + 'a {
    fn b(&self) {
        println!("I am still an A!");
    }
}
```

Because of this blanket impl, `Box<dyn A>` (and any other `dyn A + 'a`)
can call `b()` without changing the concrete type `Foo` or the traits `A`/`B`.

### UML Diagrams

#### Class Diagram

```
  ┌──────────────────────────┐          ┌──────────────────────────┐
  │       «trait» A          │          │       «trait» B          │
  │──────────────────────────│          │──────────────────────────│
  │  + a()                   │          │  + b()                   │
  └──────────────────────────┘          └──────────────────────────┘
                │ implements A                        │ implements B
                │                                     │
                │   ┌─────────────────────────────────┤
                │   │  blanket impl B for dyn A + 'a  │
                │   │  any dyn A box also gains b()   │
                ▼   ▼                                 ▼
  ┌──────────────────────────┐          ┌──────────────────────────┐
  │          Foo             │          │        dyn A + 'a        │
  │──────────────────────────│          │     «trait object»       │
  │  + new()                 │          │──────────────────────────│
  │  + a()   «impl A for Foo»│          │  + b()                   │
  │  + b()   «impl B for Foo»│          │  («via impl B for        │
  └──────────────────────────┘          │    dyn A + 'a»)          │
                                        └──────────────────────────┘
```

Trait `A` and `B` sit on top; `Foo` implements both (straight arrow plus the
crossed arm from `B`). The junction `┤` shows trait `B` also feeds the
`dyn A + 'a` trait object, giving it `b()` via the blanket impl.

#### Functions

```
────────────────────────────────────────────────────────────────────
  check_foo(item: Foo)        ──►  item.a() + item.b()           ✓ concrete type
  check_a(item: Box<dyn A>)   ──►  item.a() + item.b()           ✓ via blanket impl
  check_b(item: Box<dyn B>)   ──►  item.a() → ✗ dyn B has no a()  ↺ commented out
────────────────────────────────────────────────────────────────────
```

* `check_foo` works on the concrete type — `Foo` has both methods.
* `check_a` works on the trait object — `dyn A` gained `b()` via the blanket impl.
* `check_b` would fail: a `Box<dyn B>` only knows `b()`, so calling `a()`
  on it is a compile error (that is why this code is commented out in `main.rs`).
