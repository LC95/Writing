# RUST的内存安全保证

```rust
let author = "lc95";
```

1. **所有权树**。所有权是一个指针(或者说一个名字)的根本权力，它代表销毁自身及其子树的权力。所有权不能凭空消失，他通过一种叫做转移`move`的操作来实现转移。move语法并不改变对象的内存位置，只是从一个名字转移一个名字，转移后的对象仍然只有一个访问入口。

   1. 转移，转移意味着原有所有权树变得不完整(`partially moved`)，只有其拥有者才能够访问内容。

      ```rust
      let s = S {
          text: "hello".to_string(),
      };
      let z = s.text;//s.text已经被转移至z，所以s变得不完整了，也就无法打印了
      println!("{:?}", s);//error! borrow of partially moved value: `s`
      ```

      相同的情况也发生在数组当中

      ```rust
      let mut v = vec!["2".to_string(), "3".to_string(), "4".to_string()];
      let third = v[2];//数组，向量中所有权是永远完整的
      ```

   2. 利用转移提前终结

      ```rust
      std::mem::drop(s);
      ```

   3. 利用`Copy trait`来复制变量，默认的数字类型实现了该trait

      ```rust
      let mut v = vec![1, 2, 3];
      let third = v[2];//没有错误，因为v[2]并没有move，而是通过复制产生了一个新的所有权
      ```

2. **多所有权**

   1. 为了避免只有一个入口的不便，rust创造了引用计数`Rc<T>`和`Arc<T>`这一结构，并为该结构设置解引用规则，使得它可以像引用一样工作。

      1. 这个是通过在T的内容前面加上引用计数字段来实现的，`Rc`保存在堆上

         ```rust
         use  std::rc::Rc;
         let s0 = Rc::new("lc95".to_string());//rc=1
         {
             let s1 = s0.clone();//rc=2
             {
                 let s2 = s0.clone();//rc=3
             }//rc=2
         }//rc=1
         ```

         `s0、s1、s2`这三个都是栈上的引用，每执行`clone()`一次，都会使得引用计数+1。

         每有一个栈上的引用消失，那么就会自动使得引用计数减少。

         这是因为该类型实现了`Drop trait`，每当离开作用域的时候，自动便会执行`drop()`函数，使得引用计数减少，引用计数减到0后才销毁内容。

      2. 共享所有权可能会带来循环引用吗，前面已经分析了，move语法使得一个对象只能够有一个访问入口。`Rc`使得内容拥有多个访问入口，我们可以用它创造出两个互相引用的节点吗？

         ```rust
         struct Node {
             text: String,
             next: Option<Rc<Node>>
         }
         
         let mut node_1 = Node {
             text: "node_1".to_string(),
             next: None,
         };
         let node_1_rc = Rc::new(node_1);
         let mut node_2 = Node {
             text: "node_2".to_string(),
             next: Some(node_1_rc.clone()), //node_2 -> node_1
         };
         node_1_rc.next = Some(Rc::new(node_2));//这有可能实现吗？不行，因为rc不允许解引用为一个可修改的变量。
         //error
         //cannot assign to data in an `Rc`
         //trait `DerefMut` is required to modify through a dereference, but it is not implemented for `Rc<Node>`
         ```

         `Rc`在生成的时候就意味着其中的内容不可修改，至少现在还不行。

3. **读写权与借用**

   1. 难道我们真的需要销毁这个对象吗？如果不是的话，我们又何必要求所有权呢？我们Rust创造出借用（也就是使用权和修改权）的概念，通过`&`和`mut`标志，将一个类型变成只读/可读写的类型。我这里想强调一下类型，我们大可把所有进出小括号`()`或等号`=`的操作都叫做转移。`T`是一个类型，转移意味着读写和销毁权的转移。`&T`和`&mut T`是新的类型，称作引用；前者叫做shared reference，后者叫做mutable reference，它们的转移仅仅是读权或读写权的转移。

      1. 能够使用的引用永远不会为空

   2. RUST对这两种引用设定了规则：

      1. 读者不可写：无法对`&T`变量内部的进行修改

         ```rust
         let mut s = S {
             text: "hello".to_string(),
         };
         let read_s = &s;
         read_s.text = "aloha".to_string();//非法
         (*read_s).text = "aloha".to_string();//与上面等价
         ```

      2. 写后&写时不能用`前读`：前读，指的是写前存在的指针，指针指向的是原有位置，写后很可能原有位置的指针就悬垂了，所以写后不允许使用原有的指针

         ```rust
         let mut s = S {
             text: "hello".to_string(),
         };
         let read_s = &s;
         let mut_s = &mut s;
         mut_s.text = "aloha".to_string();
         
         //println!("{:?}", read_s);
         //本例中，我们并没有改变read_s的实际位置，但是Vec之类的扩容会改变vec的地址，如果不加以阻止就会造成悬垂指针。
         ```

         

   3. 数据结构/方法中的引用

      1. 数据结构的生命期，生命期是一个泛型参数。用表示`T<'a>`，表示数据结构中包含一个没有所有权的成员。这个数据结构消失时，是没有办法销毁该成员的。有一点是肯定的，就是引用的生命期绝对要比这个结构的生命期长。结构体中的引用不可省略生命期参数。

         ```rust
         struct Blog<'a> {
             text: &'a String,
             title:&'a String
         }
         ```

      2. 方法的生命期，所有带引用的方法均带有生命期参数，但是有时可以省略编写。

         ```rust
         fn f<'a, T1, T2>(first :&'a T1, second:&'a T2) -> &'a T1 {
             first
         }
         ```

         仅仅从函数签名上就可以看出，返回的结果将必须即满足`T1`又满足`T2`的生命期。

         ```rust
         let a = "a";
         let result;
         {
            let b = "b";
            result = f(&a, &b);
         }
         do_something(result);//不允许，result的使用超越了b的生命期。
         ```

         如何设定来避免该问题呢，我们可以让结果只与其中一个参数的生命期相关。

         ```rust
         fn f<'a, T1, T2>(first :&'a T1, second:& T2) -> &'a T1 {
             first
         }
         ```

         看参数就可以知道，方法返回的一定是`first`的一部分，它与`second`没有任何关系。

      3. 可以省略编写生命期的方法
         1. 不返回引用的方法不需要标识生命期，生命期旨在理清结果生命期与参数生命期的联系。标识结果是哪一个参数的生命期，或者需要满足所有参数的生命期。
         2. 参数中只有一个生命期，默认结果的生命期与参数的生命期一致。
         3. 参数有`&self`类型，默认结果的生命期与`self`生命期一致。
