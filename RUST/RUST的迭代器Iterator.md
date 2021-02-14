## Iterator是什么
```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    ...
}
```
```rust
let v = vec![1,2,3];
let mut iter = v.iter();
let val:i32 = a.next();//Some(&1)
iter.next();//Some(&2)
iter.next();//Some(&3)
iter.next();//None
```
可以看出Iterator是一个通过某种方式对自身进行查询并返回一个值  
某种方式指的是next方法————查询自身来确定下一个值是什么，要么Some(v)，要么None

现在有一台电视机，里面的节目就是（Iterator::Item迭代项），类似地，遥控器是一个实现了Iterator的东西，我们可以使用遥控器的next()功能来得到节目。

电视机和遥控器，一个拥有节目，一个负责找出下一个节目。除此之外没有别的联系。可是我们往往希望从电视机中（它是节目的容器）中产生一种或者几种遥控器。

我们需要一种表示某值能够产生迭代器的trait


## IntoIterator是什么？
```rust
trait IntoIterator where Self::IntoIterator::Item == Self::Item {
    type Item;
    type IntoIter:Iterator;
    fn into_iter(self) -> Self::IntoIter;
}
```
它表示·可产生迭代器的·

## for循环是什么

```rust
//Situation 1
let mut iter = tv.into_iter(); //for i in tv
while let Some(i) = iter.next() {
    //some job using i
}
//Situation 2 
while let Some(i) = iter.next() { //for i in iter
    //some job using i
}

```
for循环既可以自动将实现了IntoIterator的值转换为Iter，这一操作会消耗所有权/引用，也可以循环Iterator。

```rust
for e in & collection {} //this will consume the collection
for e in &mut collection {}
for e in & collection {}
```
