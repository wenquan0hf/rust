##type 别名

你可以使用 type 关键字声明另一类型的别名：

    type Name = String;

然后，你可以就像使用一个真正的类型一样使用这种类型：

    type Name = String;
    
    let x: Name = "Hello".to_string();

但是请注意，这是一个别名，不完全是一个新类型。换句话说，因为 Rust 是强类型的，所以你不能比较两个不同类型：

    let x: i32 = 5;
    let y: i64 = 5;
    
    if x == y {
       // ...
    }

这会产生这样的结果：

    error: mismatched types:
     expected `i32`,
    found `i64`
    (expected i32,
    found i64) [E0308]
     if x == y {
     ^

但是，如果我们有一个别名：

    type Num = i32;
    
    let x: i32 = 5;
    let y: Num = 5;
    
    if x == y {
       // ...
    }
    
这个编译没有错误。无论如何，Num 类型的值和 i32 类型的值是相同的。

你还可以使用泛型类型别名：

    use std::result;
    
    enum ConcreteError {
    Foo,
    Bar,
    }
    
    type Result<T> = result::Result<T, ConcreteError>;

这将创建一个 Result类型的专门的版本 ，它总是有一个针对 Result< T E >的 E 部分的 ConcreteError 。这常被用在标准库来为每一部分创建自定义错误。例如，io::Result 。