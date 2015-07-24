# 'Deref'强制转换

标准库提供了一个特殊的特征，Deref。它通常用于重载 * ，取消引用运算符：

    use std::ops::Deref;
    
    struct DerefExample<T> {
    value: T,
    }
    
    impl<T> Deref for DerefExample<T> {
    type Target = T;
    
    fn deref(&self) -> &T {
    &self.value
    }
    }
    
    fn main() {
    let x = DerefExample { value: 'a' };
    assert_eq!('a', *x);
    }

这用于编写自定义指针类型。然而，有一个与 Deref 相关的语言特征：‘deref 强制转换’。规则是这样的：如果你有一个类型 U，它实现 `Deref<Target=T>`，&U 的值自动强制转换为 &T。这里有一个例子：

    fn foo(s: &str) {
    // borrow a string for a second
    }
    
    // String implements Deref<Target=str>
    let owned = "Hello".to_string();
    
    // therefore, this works:
    foo(&owned);

在一个值前使用 & 需要一个引用。所以 owned 是一个 String，&owned 是一个 &String，并且由于 `impl Deref<Target=str> for String`，&String 参考传入函数 foo() 的 &str。

就这样。这条规则是 Rust 为你自动转换的少有的几处之一，但它增加了很大的灵活性。例如，类型 `Rc<T>` 实现 `Deref<Target=T>`，所以它的工作原理如下：

    use std::rc::Rc;
    
    fn foo(s: &str) {
    // borrow a string for a second
    }
    
    // String implements Deref<Target=str>
    let owned = "Hello".to_string();
    let counted = Rc::new(owned);
    
    // therefore, this works:
    foo(&counted);

所有我们所做的就是把我们的 String 封装到 `Rc<T>`。但是我们现在可以把 `Rc<String>` 传到任何有 String 的地方。foo 的声明并没有改变，但能实现与其它类型一样的功能。这个例子有两个转换：`Rc<String>` 转换为 String，然后 String 转换为 &str。Rust 会这样做尽可能多的次数直到类型匹配。　　　　

标准库提供的另一个很常见的实现是：

    fn foo(s: &[i32]) {
    // borrow a slice for a second
    }
    
    // Vec<T> implements Deref<Target=[T]>
    let owned = vec![1, 2, 3];
    
    foo(&owned);

向量可以取消对程序片的引用。

## Deref 和方法调用

Deref 调用方法时也起作用。换句话说，Rust 有相同的两件事：

    struct Foo;
    
    impl Foo {
    fn foo(&self) { println!("Foo"); }
    }
    
    let f = Foo;
    
    f.foo();

尽管 f 不是引用，但是函数 foo 中传入 &self 就会起作用。这是因为这些东西是相同的：

    f.foo();
    (&f).foo();
    (&&f).foo();
    (&&&&&&&&f).foo();

`&&&&&&&&&&&&&&&&Foo` 类型的值仍然可以有定义在 Foo 上的方法，因为编译器会插入许多 * 操作只要程序正确运行。因为它的插入* s，就要使用 Deref。