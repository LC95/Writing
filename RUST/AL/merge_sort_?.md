```rust
use std::cmp::Ord;

fn sort<T: Ord>(arr: &mut [T]) {
    if arr.len() < 2 {
        return;
    }
    let mid = arr.len() / 2;
    let (left, right) = arr.split_at_mut(mid);
    sort(left);
    sort(right);
    merge_sort_v1(left, right);
}

fn merge_sort_v1<T: Ord>(a: &mut [T], b: &mut [T]) {
    // 1,2,3,4,5,6
    // 4,5,6,7,8,9
    let mut i = 0;
    let mut j = 0;
    while i < a.len() && j < b.len() {
        if a[i] < b[j] {
            i += 1;
        } else {
            std::mem::swap(&mut a[i], &mut b[j]);
            j += 1;
        }
    }
}
```
应该是归并xD