# 闭包

Rust 不仅命名函数，还命名匿名函数。匿名函数有一个关联的环境被称为“闭包”，因为他们关闭了一个环境。如我们将会看到的，Rust 对它们有完美的实现。 

## 语法

闭包看起来是这样的：
    
    let plus_one = |x: i32| x + 1;
    
    assert_eq!(2, plus_one(1));

我们创建一个绑定，plus_one 并将其分配给一个闭包。关闭管道之间的参数(|)，并且主体是一个表达式，在本例中是 x + 1。记住{ }也是一个表达式，所以我们可以多行的闭包:
    
    let plus_two = |x| {
    let mut result: i32 = x;
    
    result += 1;
    result += 1;
    
    result
    };
    
    assert_eq!(4, plus_two(2));

你会注意到闭包与用 fn 定义的普通函数略有不同。第一个不同点是，我们不需要对参数的类型和返回值进行注释。我们可以：

    let plus_one = |x: i32| -> i32 { x + 1 };
    
    assert_eq!(2, plus_one(1));

但我们并不需要这样做。这是为什么呢?根本上来说，这是出于人体工程学的考虑。对于文档和类型推断而言，指定已命名函数的完整类型命名是有用的，然而闭包的类型很少被记录因为他们是匿名的，而且他们不会引 error-at-a-distance 错误，这种错误能够推断命名函数的类型。

第二，语法是相似的，但有点不同。我在这里添加空格让他们看起来更接近：

    ffn  plus_one_v1   (x: i32 ) -> i32 { x + 1 }
    let plus_one_v2 = |x: i32 | -> i32 { x + 1 };
    let plus_one_v3 = |x: i32 |  x + 1  ;

差异很小，但它们类似。

## 闭包和他们的环境

闭包之所以被称为闭包，是因为他们封闭了他们的环境。它看起来像这样：
    
    let num = 5;
    let plus_num = |x: i32| x + num;
    
    assert_eq!(10, plus_num(5));

plus_num，这个闭包指的是一个 let 绑定在它的范围 num 内。更具体地说，它借用了绑定。如果我们做与绑定相冲突的事情，就会得到一个错误。像下面这样： 

    let mut num = 5;
    let plus_num = |x: i32| x + num;
    
    let y = &mut num;

错误如下：

    error: cannot borrow `num` as mutable because it is also borrowed as immutable
    let y = &mut num;
     ^~~
    note: previous borrow of `num` occurs here due to use in closure; the immutable
      borrow prevents subsequent moves or mutable borrows of `num` until the borrow
      ends
    let plus_num = |x| x + num;
       ^~~~~~~~~~~
    note: previous borrow ends here
    fn main() {
    let mut num = 5;
    let plus_num = |x| x + num;
    
    let y = &mut num;
    }
    ^

这里的错误信息虽然冗长却是有用的!比如说，我们不能对 num 进行一个可变的 borrow，因为闭包已经 borrow 过它了。如果我们让闭包超出范围，我们可以：

    let mut num = 5;
    {
    let plus_num = |x: i32| x + num;
    
    } // plus_num goes out of scope, borrow of num ends
    
    let y = &mut num;

如果你的闭包需要它，Rust 将获取所有权并且移动环境：
    
    let nums = vec![1, 2, 3];
    
    let takes_nums = || nums;
    
    println!("{:?}", nums);

这告诉我们：
    
    note: `nums` moved into closure environment here because it has type
      `[closure(()) -> collections::vec::Vec<i32>]`, which is non-copyable
    let takes_nums = || nums;
    ^~~~~~~

`Vec < T >` 对其内容拥有所有权，因此，当我们在闭包的操作涉及到它时，我们将不得不声称对 num 的所有权。同样的，如果我们将 num 传递给一个函数，则这个函数对其拥有所有权。

## 移动闭包

我们可以强制我们的闭包用 move 关键字获取环境移所有权：
    
    let num = 5;
    
    let owns_num = move |x: i32| x + num;

现在，即使关键字是 move，变量仍然遵循正常 move语义。在这种情况下，5 实现复制，所以 owns_num 持有 num 的副本。然而区别在哪里呢?

    let mut num = 5;
    
    { 
    let mut add_num = |x: i32| num += x;
    
    add_num(5);
    }
    
    assert_eq!(10, num);

所以在这种情况下，我们的闭包获得了一个可变 num，我们称为 add_num， 如我们所期望的，它改变了 num 的潜在值。我们还需要将 add_nu m声明为 mut，因为我们正在改变其环境。

如果我们换成一个 move 闭包，就会出现不同：
    
    let mut num = 5;
    
    { 
    let mut add_num = move |x: i32| num += x;
    
    add_num(5);
    }
    
    assert_eq!(5, num);

我们只得到 5。而不是从 num 得到可变的 borrow 我们对副本拥有所有权。

另一种方式思考 mov e闭包:他们分配给闭包一个自己的堆栈帧。没有 move 一个闭包可能与创建它的堆栈帧联系到一起，而且闭包是自包含的。这意味着你不能从函数返回一个 non-move 闭包。

但在我们讨论使用和返回闭包并之前，我们应该更多的讨论闭包的实现方式。作为一种系统语言、Rust 让你能够控制代码所做的事情，而闭包是没有什么不同的。 

## 闭包实现

Rust 的闭包实现有点不同于其他语言。对特征来说他们是非常高效的语言。你要确保阅读这一章之前已经阅读了特征这一章，以及特征对象这一章。

都明白了吗？很棒。

闭包工作的关键点有些奇怪:使用()调用一个函数，就像 foo() 是一种可重载操作符。由此，在其他的任何地方单击鼠标都能进入空间。在 Rust 语言里面，我们使用特征系统重载操作符。调用函数也不例外。我们有三个独立的过载与特征：

    pub trait Fn<Args> : FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
    }
    
    pub trait FnMut<Args> : FnOnce<Args> {
    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
    }
    
    pub trait FnOnce<Args> {
    type Output;
    
    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
    }
    
你会注意到这些特征之间的差异，但很大的一个点是 self:Fn 和 &self,FnMut 和 &mut self,FnOnce 和 self。这通过常用的方法调用语法涵盖了所有三种 self。但是我们把他们分成三个特征，而不是一个。这给了我们足够的控制权去决定我们可以采用什么样的闭包。

对闭包的| | { }语法对这三个特征来说是糖衣语法。Rus t将为环境生成一个 struct， impl 适当的特征,然后使用它。

## 使用闭包作为参数

现在我们知道，闭包特征，我们已经知道如何接受和返回闭包:就像任何其他特征那样!

这也意味着我们可以选择静态与动态调度。首先，让我们写一个可以调用其他函数的函数，调用它，并返回结果：

    fn call_with_one<F>(some_closure: F) -> i32
    where F : Fn(i32) -> i32 {
    
    some_closure(1)
    }
    
    let answer = call_with_one(|x| x + 2);
    
    assert_eq!(3, answer);

我们通过闭包，`| | x + 2`去调用 call_with_one。它只是做它所表明的事情：它调用闭包，1 作为参数。

让我们更深入地检查 call_with_one 的签名：

    fn call_with_one < F >(some_closure:F)- >i32

我们需要一个参数，它的类型为 F .我们也返回一个 i32。这部分不是很有趣。下一个部分是：

    where F : Fn(i32) -> i32 {

因为 Fn 是一个特征，我们可以将我们的泛型和它绑定到一起。在这种情况下，我们的闭包需要把 i32 作为参数，并返回一个 i32 所以我们使用的泛型边界是 `Fn(i32) -> i32`。

这里还有一个关键问题:因为我们吧泛型和特征绑定到了一起，这将导致单形态，因此，我们将在闭包里面做静态调度。那是就清晰多了。在许多语言中，闭包本身就是堆分配，总是涉及到动态调度。在 Rust 语言中，我们可以用堆栈分配我们的闭包环境，和静态调度 call 语句。这种情况通常发生于迭代器和适配器，通常采用闭包作为参数。

当然，如果我们想要动态调度，我们也可以那样。特征对象处理这种情况时，像往常一样：
    
    fn call_with_one(some_closure: &Fn(i32) -> i32) -> i32 {
    some_closure(1)
    }
    
    let answer = call_with_one(&|x| x + 2);
    
    assert_eq!(3, answer);
    
现在我们把一个特征对象,`&Fn`。当我们传递这个特征到 call_with_one 时我们必须参考我们的闭包,所以我们使用 `& | |`。

## 返回闭包
    
在各种情况下，返回闭包是很常见的函数形式。如果你想返回一个闭包，您可能会遇到一个错误。起初，这看起来可能有些奇怪，但是我们会找到答案。你可以试着从下面的函数返回一个闭包：
    
    fn factory() -> (Fn(i32) -> Vec<i32>) {
    let vec = vec![1, 2, 3];
    
    |n| vec.push(n)
    }
    
    let f = factory();
    
    let answer = f(4);
    assert_eq!(vec![1, 2, 3, 4], answer);
    error: the trait `core::marker::Sized` is not implemented for the type
    `core::ops::Fn(i32) -> collections::vec::Vec<i32>` [E0277]
    f = factory();
    ^
    note: `core::ops::Fn(i32) -> collections::vec::Vec<i32>` does not have a
    constant size known at compile-time
    f = factory();
    ^
    error: the trait `core::marker::Sized` is not implemented for the type
    `core::ops::Fn(i32) -> collections::vec::Vec<i32>` [E0277]
    factory() -> (Fn(i32) -> Vec<i32>) {
     ^~~~~~~~~~~~~~~~~~~~~
    note: `core::ops::Fn(i32) -> collections::vec::Vec<i32>` does not have a constant size known at compile-time
    fa ctory() -> (Fn(i32) -> Vec<i32>) {
      ^~~~~~~~~~~~~~~~~~~~~

为了从一个函数返回一些东西，Rust 需要知道返回类型的大小。但由于 Fn 是一个特征，它可能具有是各种不同大小的 `size :` 许多不同的类型都可以实现 Fn。来给一些事物赋予 siz e的一种简单的方法是参考已知的 size。所以如下面我们写的这样：

    fn factory() -> &(Fn(i32) -> Vec<i32>) {
    let vec = vec![1, 2, 3];
    
    |n| vec.push(n)
    }
    
    let f = factory();
    
    let answer = f(4);
    assert_eq!(vec![1, 2, 3, 4], answer);
    
但我们得到了另一个错误：

    error: missing lifetime specifier [E0106]
    fn factory() -> &(Fn(i32) -> i32) {
    ^~~~~~~~~~~~~~~~~
    
正确的。因为我们有一个参考，我们需要给它指定生命周期。但是我们的 factory() 函数不带参数，所以此处省略不写。我们可以选择什么样的生命周期呢?静态：

    
    fn factory() -> &'static (Fn(i32) -> i32) {
    let num = 5;
    
    |x| x + num
    }
    
    let f = factory();
    
    let answer = f(1);
    assert_eq!(6, answer);

但我们得到另一个错误：
    
    error: mismatched types:
     expected `&'static core::ops::Fn(i32) -> i32`,
    found `[closure <anon>:7:9: 7:20]`
    (expected &-ptr,
    found closure) [E0308]
     |x| x + num
     ^~~~~~~~~~~

这个错误是让我们知道，我们没有 `&'static Fn(i32) -> i32` 但是我们有一个`[closure <anon>:7:9: 7:20]`。等等，这是什么?

因为每个闭包生成自己的环境结构和实现 Fn 特征以及 friends 这些都是匿名的。它们的存在只是因为这个闭包。Rust 把他们显示为 `closure <anon>`而不是一些自动生成的名字。

但是为什么我们的闭包不实现 `&'static Fn `?正如我们之前讨论的，闭包  borrow 了他们的环境。在这种情况下，我们的环境是基于 5 栈分配以及num 变量绑定。因此，borrow 的生命周期为的堆栈帧长度。所以如果我们返回这个闭包，函数调用将结束，堆栈框架将消失，并且我们的闭包将对垃圾内存的环境进行捕获!

那么该怎么办?这就是工作原理：

    fn factory() -> Box<Fn(i32) -> i32> {
    let num = 5;
    
    Box::new(|x| x + num)
    }
    let f = factory();
    
    let answer = f(1);
    assert_eq!(6, answer);
    
我们使用一个特征对象,通过建立 Fn。只有最后一个问题：

    error: `num` does not live long enough
    Box::new(|x| x + num)
     ^~~~~~~~~~~
    
我们仍然有一个对父堆栈帧的引用。通过最后一次修复，我们这样做可以做可以行得通：

    fn factory() -> Box<Fn(i32) -> i32> {
    let num = 5;
    
    Box::new(move |x| x + num)
    }
    let f = factory();
    
    let answer = f(1);
    assert_eq!(6, answer);
    
通过内部 move Fn，我们可以为我们的闭包创建一个新的堆栈帧。我们给它一个已知大小，并对它进行填充，而且允许他脱离我们的堆栈帧。

