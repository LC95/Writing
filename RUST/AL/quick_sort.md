## 整数版本
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

## 泛型版本
```rust
use std::cmp::Ord;
fn quick_sort<T: Ord>(arr: &mut [T]) {
    if arr.len() <= 1 {
        return;
    }
    let mid = sort_by_mid(arr);
    quick_sort(&mut arr[..mid]);
    quick_sort(&mut arr[mid + 1..]);
}
fn sort_by_mid<T: Ord>(arr: &mut [T]) -> usize {
    let (mut i_left, mut i_right) = (0, arr.len() - 1);
    while i_left < i_right {
        //此时，mid值在左边即arr[i_left]
        while arr[i_right] >= arr[i_left] && i_left < i_right {
            i_right -= 1;
        }
        arr.swap(i_left, i_right);
        //此时，mid值被换到了右边即arr[i_right]
        while arr[i_left] <= arr[i_right] && i_left < i_right {
            i_left += 1;
        }
        //此时，mid值又被换到了左边
        arr.swap(i_left, i_right);
    }
    return i_left;
}
```
注意整数版本的倒数第二个语句`arr[i_left] = mid_number`，考察一下该语句在泛型版本中需要做出的改变
mid_number从`u32`变为`&T`
`arr.set(i_left, mid_number)`可行吗?
不行，编译器说为set需要的是`T`而不是`&T`
那么这样呢？
`arr.set(i_left, *mid_number);`
这样也不行，编译器说`mid_number:&T`是一个对`arr`的Immutable borrow，现在不能使用`arr`的mutable borrow。
赋值时，持有任何一个arr的引用，都使得对arr的赋值变得不可能。

`arr[left] = arr[right]`这样不行，如果元素T没有实现Copy Trait的话。

sln：
简单地，如果不bind arr的当中值的引用，拿不到一个mid的引用，这样就需要知道mid的实际位置，如上边注释所写的，mid有时在i_left，有时在i_right，悲剧的是，我们似乎只能通过数组内交换的方法来实现。

如果数组内元素都是Option<T>那就好办了，值可以拿出来，并且替换地放进去一个None。最后在那个None的位置填上值。

其他方法，我不知道