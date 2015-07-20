##Borrow 和 AsRef##

[Borrow](https://doc.rust-lang.org/stable/std/borrow/trait.Borrow.html) 和 [AsRef](https://doc.rust-lang.org/stable/std/convert/trait.AsRef.html) 特性是非常相似的，但是也有些区别。这里有一个简单回顾一下这两个特质是什么意思。

###Borrow###

Borrow 特性是当你写一个数据结构时，并且你想要使用一个 owned 或 borrowed 类型作为用于某些目的的同义词。

例如，HashMap 的 get 方法就使用了 Borrow:

```
 fn get<Q: ?Size>(&self, k:&Q) -> Option<&V>
	where K:Borrow<Q>,
		  Q:Hash +Eq
```

这个签名是相当复杂的。在这里我们感兴趣的是 K 参数。它指的是 HashMap 本身的一个参数:

```
struct HashMap<K, V, S = RandomState> {
```

K 参数是 HashMap 中使用的 key 类型。所以，再次看 get() 函数签名，当 key 实现了 Borrow<Q> 时我们可以使用 get() 函数。这样，我们可以使用 String 类型作为 HashMap 的 key，而使用 &strs 进行搜索:

```
use std::collections::HashMap:

let mut map = HashMap::new();
map.insert("Foo".to_string(), 42);

assert_eq!(map.get("Foo"), Some(&42));
```

这是因为标准库中已经为 String 类型实现了 Borrow<str> 接口。

对于大多数类型，当你想要一个使用 owned 或者 borrowed 类型，使用 &T 就足够了。但在不止一种 borrowed 值时，Borrow 处理起来是很高效的。Slice 指的是一块区域：既可以是 &[T]也可以是 &mut[t] 类型。如果我们想使该区域同时存在着两种类型，就可以使用 Borrow:

```
use std::borrow::Borrow;
use std::fmt::Display;

fn foo<T: Borrow<i32> + Display>(a: T) {
    println!("a is borrowed: {}", a);
}

let mut i = 5;

foo(&i);
foo(&mut i);
```

上面的代码会输出 a is borrowed: 5 两次。

###AsRef###

AsRef 是转换特征。它被用在通用的代码中将一些值转换成索引类型。比如：

```
let s = "Hello".to_string();

fn foo<T: AsRef<str>>(s: T) {
	let slice = s.as_ref();
}
```

###应该用哪一个？###

我们可以看到他们是如何的相似：他们都处理 owned 和 borrowed 版本的数据类型。然而，他们还是有点不同。

当你想要抽象出各种 borrowing 数据中的不同，或者你创建的数据结构用同样的方式对待 owned 和 borrowed 值，比如哈希操作和比较操作，此时你应该选择 Borrow。

当你想要直接转换某些类型为引用类型，并且你在编写泛型时，你应该选择 AsRef。
