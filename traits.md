# 特征

你还记得 impl 关键字吗，它用于调用一个函数的 [methodsyntax](http://doc.rust-lang.org/stable/book/method-syntax.html)

    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl Circle {
    fn area(&self) -> f64 {
    std::f64::consts::PI * (self.radius * self.radius)
    }
    }

特征几乎都是相似的，除了我们用方法签名去定义一个特征,然后实现该结构的特征。如下面所示：
    
    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    trait HasArea {
    fn area(&self) -> f64;
    }
    
    impl HasArea for Circle {
    fn area(&self) -> f64 {
    std::f64::consts::PI * (self.radius * self.radius)
    }
    }
    
如您所见，这个特征块和 impl 块非常相似，但我们不定义一个主体，只是定义一个类型签名。当我们 impl 一个特征时，我们使用 impl 特征项，而不仅仅是 impl 项。

我们可以使用特征约束泛型。思考下面这个函数，它没有编译，只给我们一个类似的错误：
    
    fn print_area<T>(shape: T) {
    println!("This shape has an area of {}", shape.area());
    }
    
Rust 可能会抱怨道：

    error: type `T` does not implement any method in scope named `area`

因为 T 可以是任何类型，我们不能确保它实现了 area 的方法。但我们可以添加一个特征约束的泛型 T，确保它已经实现：

    fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
    }

语法 `< T:HasArea >` 意味着实现 HasArea 特征的任何类型。因为特征定义函数的类型签名，我们可以肯定，任何实现 HasArea 的类型都会有.area()方法。

这里有一个扩展的例子展示它是如何工作的：

    trait HasArea {
    fn area(&self) -> f64;
    }
    
    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl HasArea for Circle {
    fn area(&self) -> f64 {
    std::f64::consts::PI * (self.radius * self.radius)
    }
    }
    
    struct Square {
    x: f64,
    y: f64,
    side: f64,
    }
    
    impl HasArea for Square {
    fn area(&self) -> f64 {
    self.side * self.side
    }
    }
    
    fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
    }
    
    fn main() {
    let c = Circle {
    x: 0.0f64,
    y: 0.0f64,
    radius: 1.0f64,
    };
    
    let s = Square {
    x: 0.0f64,
    y: 0.0f64,
    side: 1.0f64,
    };
    
    print_area(c);
    print_area(s);
    }

这个程序输出：

    This shape has an area of 3.141593
    This shape has an area of 1

如您所见，print_area 现在是通用的，同时也确保我们通过了正确的类型。如果我们通过一个错误的类型：

    print_area(5);

我们会得到一个编译错误：

    error: failed to find an implementation of trait main::HasArea for int

到目前为止，我们只添加特征实现结构，但您可以实现任何类型的特征。所以从技术上讲，我们可以为 i32 实现 HasArea：
    
        trait HasArea {
    fn area(&self) -> f64;
    }
    
    impl HasArea for i32 {
    fn area(&self) -> f64 {
    println!("this is silly");
    
    *self as f64
    }
    }
    
    5.area();

我们一般认为用这种基本类型来实现方法是一种不够好的风格，即便这种实现方法是可行的。

这可能看起来比较粗糙，但是有两个其他的限制来控制特征的实现,防止失控。首先，如果特征不是在你的范围中定义的,那么不适用。下面是一个例子：标准库提供了编写特征，这个特征给文件的输入输出增加了额外的功能。默认情况下，文件并不会有自己的方法：

    let mut f = std::fs::File::open("foo.txt").ok().expect("Couldn’t open foo.txt");
    let result = f.write("whatever".as_bytes());

这里有一个错误：

    error: type `std::fs::File` does not implement any method in scope named `write`
    
    let result = f.write(b"whatever");

       ^~~~~~~~~~~~~~~~~~

我们需要先使用编写特征：

    use std::io::Write;
    
    let mut f = std::fs::File::open("foo.txt").ok().expect("Couldn’t open foo.txt");
    let result = f.write("whatever".as_bytes());

此时编译没有出现错误。

这意味着，即使有人做了坏事比如向 int 添加方法，也不会影响你，除非你使用这个特征。

实现特征还有其他限制。你为特征或类型所写的 impl 必须由你来定义。所以,我们可以实现 HasArea 类型等，因为 HasArea 是我们的代码。但是如果我们试图实现浮点型,它是 Rust 为 i32 所提供的特征，是不可能的，因为无论特征还是类型都不在我们的代码里面。

最后关于特征：通用函数特征的捆绑使用 “monomorphization”(mono:一个，morph:形式),所以他们都是静态调用。那意味着什么呢?到特征对象那一章查看更多细节。

## 多个特征边界

如您所看到的，您可以绑定特征与一个泛型的类型参数：

    fn foo<T: Clone>(x: T) {
    x.clone();
    }

如果你需要多个绑定，您可以使用+：

    use std::fmt::Debug;
    
    fn foo<T: Clone + Debug>(x: T) {
    x.clone();
    println!("{:?}", x);
    }
    
T 现在需要复制以及调试。

## Where 子句

只用少数泛型和少量的特征界限来写函数并不是太糟糕，但随着数量的增加，语法变得越来越糟糕：
    
    use std::fmt::Debug;
    
    fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
    }
    println !(“{:?}”,y);
    }
    
函数的名称是在最左边，参数列表在最右边。边界就以这种方式存在。

Rust 已经有了解决方法，这就是所谓的“where 子句”：

    use std::fmt::Debug;
    
    fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
    }
    
    fn bar<T, K>(x: T, y: K) where T: Clone, K: Clone + Debug {
    x.clone();
    y.clone();
    println!("{:?}", y);
    }
    
    fn main() {
    foo("Hello", "world");
    bar("Hello", "workd");
    }

foo()使用我们之前讲解的语法，bar()使用一个 where 子句。所有你需要做的就是定义类型参数时不要定义边界，然后在参数列表之后添加 where 语句。对于更长的列表，可以添加空格：

    use std::fmt::Debug;
    
    fn bar<T, K>(x: T, y: K)
    where T: Clone,
      K: Clone + Debug {
    
    x.clone();
    y.clone();
    println!("{:?}", y);
    }

这种灵活性可以使复杂情况变得清晰。

Where 语句也比简单的语法更加强大。例如：

    trait ConvertTo<Output> {
    fn convert(&self) -> Output;
    }
    
    impl ConvertTo<i64> for i32 {
    fn convert(&self) -> i64 { *self as i64 }
    }
    
    // can be called with T == i32
    fn normal<T: ConvertTo<i64>>(x: &T) -> i64 {
    x.convert()
    }
    
    // can be called with T == i64
    fn inverse<T>() -> T
    // this is using ConvertTo as if it were "ConvertFrom<i32>"
    where i32: ConvertTo<T> {
    1i32.convert()
    }

这个例子展了示 where 子句的附加特性：它们允许范围内的左边可以是一个任意的类型(在这里是 i32)，而不只是一个普通的类型参数(如 T)。

## 默认的方法

我们的最后一个特征的特性应包括：默认的方法。举个简单的例子更容易说明：

    trait Foo {
    fn bar(&self);
    
    fn baz(&self) { println!("We called baz."); }
    }

Foo 特征的实现者需要实现 bar()方法，但是他们不需要实现 baz()。他们会得到这种默认行为。如果他们这么选择的话就可以覆盖默认的情形：

    struct UseDefault;
    
    impl Foo for UseDefault {
    fn bar(&self) { println!("We called bar."); }
    }
    
    struct OverrideDefault;
    
    impl Foo for OverrideDefault {
    fn bar(&self) { println!("We called bar."); }
    
    fn baz(&self) { println!("Override baz!"); }
    }
    
    let default = UseDefault;
    default.baz(); // prints "We called baz."
    
    let over = OverrideDefault;
    over.baz(); // prints "Override baz!"
    ⇱
    
## 继承

有时，想要实现某个特征需要先实现另一个特征：
    
    trait Foo {
    fn foo(&self);
    }
    
    trait FooBar : Foo {
    fn foobar(&self);
    }
     FooBar的实现者也要实现Foo,像这样:
    struct Baz;
    
    impl Foo for Baz {
    fn foo(&self) { println!("foo"); }
    }
    
    impl FooBar for Baz {
    fn foobar(&self) { println!("foobar"); }
    }

如果我们忘记实现 Foo，Rust 会告诉我们：

    error: the trait `main::Foo` is not implemented for the type `main::Baz` [E027]
  




































