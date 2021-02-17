```rust
let a = &arr[0]; //元素的引用

let num = numbers[0]; //Copy
let c = arr[0].clone(); //Clone
let my_ref = &buffer[4..12];
let my_copy = buffer[4..12].to_vec(); //Clone
```

使用`u32` `u64` `isize`作为索引会出错，只能使用`usize`

```rust
slice.first(); //or slice.first_mut();
slice.last(); //or slice.last_mut();
slice.get(index); //or slice.get_mut(index);
```