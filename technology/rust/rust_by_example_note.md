# NOTE

## Links

- [Rust by Example](https://doc.rust-lang.org/book/ch10-00-rust-by-example.html)

## tuple

- Underscores can be inserted in numeric literals to improve readability, e.g. 1_000 is the same as 1000, and 0.000_001 is the same as 0.000001.

- tuple 没限制只能存 2 个元素，而且元素的类型可以不同。严格来说，你也能创建一个元素的 tuple，但必须为那一个元素加上逗号

```rust
let oneElementTuple = (5u32,);
```

- tuple 支持类型 groovy，kotlin 那样的 binding

```rust 
let tuple = (1, "hello", 3, true);
let (a, b ,c ,d) = tuple;
```

## slice

slice has no idea about length, they just have two words. The first one is pointer to data, the second word is the length of the slice.

## Struct

three type structs

- Tuple struct
- classic C struct
- Unit struct

```Rust
// Tuple struct
struct Pair(i32, f32);

// Unit struct
struct Unit;

 let pair = Pair(1, 0.1);
 println!("pair contains {:?} and {:?}", pair.0, pair.1);

 let _unit = Unit;
```

## Enums

```rust
enum WebEvent {
    // An `enum` may either be `unit-like`,
    PageLoad,
    PageUnload,
    // like tuple structs,
    KeyPress(char),
    Paste(String),
    // or c-like structures.
    Click { x: i64, y: i64 },
}
```

