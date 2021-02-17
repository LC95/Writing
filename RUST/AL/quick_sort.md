```rust
fn quick_sort(arr: &mut [u32]) {
    if arr.len() <= 1 {
        return;
    }
    let mid = sort_by_mid(arr, arr[0]);
    quick_sort(&mut arr[..mid]);
    quick_sort(&mut arr[mid + 1..]);
}

fn sort_by_mid(arr: &mut [u32], mid_number: u32) -> usize {
    let mut i_left = 0;
    let mut i_right = arr.len() - 1;
    while i_left < i_right {
        while arr[i_right] >= mid_number && i_left < i_right {
            i_right -= 1;
        }
        arr[i_left] = arr[i_right];
        while arr[i_left] <= mid_number && i_left < i_right {
            i_left += 1;
        }
        arr[i_right] = arr[i_left];
    }
    arr[i_left] = mid_number;
    return i_left;
}
```