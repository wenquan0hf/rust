# 宏命令

现在你已经了解了很多 Rust 为抽象和重用代码提供的工具。这些代码重用的单元有丰富的语义结构。例如，函数有一个类型声明，有特征约束的类型参数，重载的函数必须属于一个特定的特征。　　　　

这种结构意味着 Rust 的核心抽象有强大的编译时正确性检查。但这是以灵活性的减少为代价。如果你从表面上识别重复代码的模式，你可能会发现像一个泛型函数，特征，或者 Rust 语义中其它任何东西一样表达模式是很困难的或者是很繁琐的。

宏定义允许我们实现语法水平上的抽象。宏调用的简单来说就是“扩大”语法形式。这种扩张发生在编译早期，在任何静态检查之前。因此，宏可以捕获许多代码重用模式，这些是 Rust 的核心抽象做不到的。　　　　

缺点是基于宏的代码比较难以理解,因为更少的内置规则可以使用。像一个普通的函数，可以使用一个功能良好的宏而无需理解它的实现。然而，很难设计一个功能良好的宏！此外，在宏代码中的编译错误难以解释，因为他们用扩展代码来描述问题，而不是开发人员使用的源代码级别的形式。

这些缺点是宏的重要的“特性”。这并不是说宏不好，它是有时是 Rust 的一部分，因为他们真正需要简洁、抽象的代码。我们要记住这个折衷。

## 定义一个宏

你可能看到过宏 vec!，用于初始化包含任意数量元素的向量。

    let x: Vec<u32> = vec![1, 2, 3];

这不可能是一个普通的函数，因为它有任意数量的参数。但我们可以把它想象成下面语法的简称

    let x: Vec<u32> = {
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
    };

我们可以使用一个宏实现这个函数：

    macro_rules! vec {
    ( $( $x:expr ),* ) => {
    {
    let mut temp_vec = Vec::new();
    $(
    temp_vec.push($x);
    )*
    temp_vec
    }
    };
    }

哇，这是一个新的语法！让我们分解一下。

    macro_rules! vec { ... }

这表示我们定义一个名为 vec 的宏，正如 fn vec 将定义一个名为 vec 的函数。换句话说，我们用一个感叹号非正式地编写一个宏的名字，例如 vec!。感叹号是调用语法的一部分，用来区分一个宏和一个普通的函数。

## 匹配

宏是通过一系列的规则来定义的，这些规则是用来模式匹配的。上面，我们有

    ( $( $x:expr ),* ) => { ... };

这就像一个匹配表达式的处理器，但编译时匹配发生 Rust 语法树。在最后的实例后面分号是可选的。= > 左边”的“模式”被称为“匹配器”。这些在语言里都有自己的小语法。　　　　

匹配器 $x:expr 通过将语法树绑定到元变量 $x 来匹配任何 Rust 表达式。标识符 expr 是一个片段说明符，完整的标识符是在本章后面的枚举。`$(...),*` 周围的匹配器将匹配零个或多个表达式，这些表达式由逗号分隔开。　　　　

除了特殊的匹配器语法，任何出现在一个匹配器中的 Rust 指令必须完全匹配。例如，

    macro_rules! foo {
    (x => $e:expr) => (println!("mode X: {}", $e));
    (y => $e:expr) => (println!("mode Y: {}", $e));
    }
    
    fn main() {
    foo!(y => 3);
    }

会打印出

    mode Y: 3

通过函数

    foo!(z => 3);

我们会得到以下编译错误

    error: no rules expected the token `z`

## 扩展

在大多数情况下，宏观规则的右边是普通的 Rust 语法。但我们可以拼接一些被匹配器捕获的语法。下面是一个典型的例子：

    $(
    temp_vec.push($x);
    )*

每个匹配表达式 $x 在宏扩展中产生一个单独的 push 语句。扩展的副本与匹配器中的副本是同步的。　　　　

因为 $x 已经声明为一个表达式匹配，我们不要在右侧重复 :expr。同样，我们不能把逗号作为重复操作符的一部分。相反，我们在重复的块内有一个终止分号。　　　　

另一个细节：vec! 宏在右侧有两个双括号。他们通常组合如下：

    macro_rules! foo {
    () => {{
    ...
    }}
    }

外层的括号是语法 macro_rules! 的一部分。实际上，你也可以使用 () 或 []。他们只是将右侧划分为一个整体。　　　　

内层括号是扩展语法的一部分。记住，vec! 宏被用在一个表达式上下文。为了写一个包含多个语句的表达式，包括 let-bindings，我们需要使用一个块。如果你的宏扩展到一个单个表达式，你就不需要这些额外的括号。　　　　

注意，我们从未声明宏产生一个表达式。事实上，这是不确定的，直到我们作为一个表达式使用宏。小心，你可以编写一个宏，它的扩展可以在多个上下文中起作用。例如，数据类型的简写作为一个表达式或模式是有效的。

## 重复

重复操作符遵循以下两个主要规则：

1. `$(...)*` 为它包含的所有 $name 同步处理一个重复“层”，并且
2. 每个 $name 必须至少在它能匹配的尽可能多的 `$(...)*` 下。如果它在更多的重复操作符下，它会适当的复制。

这个结构复杂的宏说明了变量从外层重复层的复制。

    macro_rules! o_O {
    (
    $(
    $x:expr; [ $( $y:expr ),* ]
    );*
    ) => {
    &[ $($( $x + $y ),*),* ]
    }
    }
    
    fn main() {
    let a: &[i32]
    = o_O!(10; [1, 2, 3];
       20; [4, 5, 6]);
    
    assert_eq!(a, [11, 12, 13, 24, 25, 26]);
    }

上面包含了大部分的匹配器语法。这些例子使用 `$(...)*` ，这是一种“零个或多个”匹配。或者你可以写 `$(...)+` 进行“一个或多个”匹配。两种形式都可选地包括一个分隔符，它可以是任何除 `+` 或 `*` 的符号。　　　　

该系统是基于“Macro-by-Example”的。

## 卫生

一些语言通过使用简单的文本替换来实现宏，从而导致各种各样的问题。例如，这个 C 程序打印 13，而不是预期的 25。

    #define FIVE_TIMES(x) 5 * x
    
    int main() {
    printf("%d\n", FIVE_TIMES(2 + 3));
    return 0;
    }

扩展后，我们有 5 * 2 + 3，乘法有比加法更高的优先级。如果你使用很多 C 宏，你可能知道以避免这个问题的通用方法，还有其它五六种方法。在 Rust 里，我们不必担心这些。

    macro_rules! five_times {
    ($x:expr) => (5 * $x);
    }
    
    fn main() {
    assert_eq!(25, five_times!(2 + 3));
    }

元变量 $x 被解析为一个表达式节点，并保持它在语法树上的位置，即使在替换以后。　　　　

宏观系统的另一个常见的问题是“变量捕获”。这里有一个 C 宏，使用 a GNU C extension模拟 Rust 的表达式块。
    
    #define LOG(msg) ({ \
    int state = get_log_state(); \
    if (state > 0) { \
    printf("log(%d): %s\n", state, msg); \
    } \
    })

这是一个简单发生严重故障的用例：

    const char *state = "reticulating splines";
    LOG(state)

这可以扩展为

    const char *state = "reticulating splines";
    int state = get_log_state();
    if (state > 0) {
    printf("log(%d): %s\n", state, state);
    }

命名为 state 第二个变量覆盖了第一个变量。这是一个问题,因为 print 语句应该参考这两个变量。　　　　

下面 Rust 宏可以达到预期结果。

    macro_rules! log {
    ($msg:expr) => {{
    let state: i32 = get_log_state();
    if state > 0 {
    println!("log({}): {}", state, $msg);
    }
    }};
    }
    
    fn main() {
    let state: &str = "reticulating splines";
    log!(state);
    }

这个能起作用是因为 Rust 有一个卫生宏系统。每个宏扩展发生在一个独特的“语法语境”，每个变量被产生它的语法语境所标记。好像 main 里面的变量 state 在宏里被涂上不同的颜色，因此他们互相不冲突。　　　　

这也限制了宏在调用点引入新的绑定的能力。以下代码就不会起作用：

    macro_rules! foo {
    () => (let x = 3);
    }
    
    fn main() {
    foo!();
    println!("{}", x);
    }

相反，你需要通过变量名调用，所以它被正确的语法语境所标记。

    macro_rules! foo {
    ($v:ident) => (let $v = 3);
    }
    
    fn main() {
    foo!(x);
    println!("{}", x);
    }

这适用于 let 绑定和循环标签，而不适用于 items。那么下面的代码可以通过编译：

    macro_rules! foo {
    () => (fn x() { });
    }
    
    fn main() {
    foo!();
    x();
    }

## 递归宏

一个宏的扩展可以包括更多的宏调用，包括调用正在扩展的同一宏。这些递归宏用于处理树形结构输入，正如下面(简单的) HTML 速记所示：

    macro_rules! write_html {
    ($w:expr, ) => (());
    
    ($w:expr, $e:tt) => (write!($w, "{}", $e));
    
    ($w:expr, $tag:ident [ $($inner:tt)* ] $($rest:tt)*) => {{
    write!($w, "<{}>", stringify!($tag));
    write_html!($w, $($inner)*);
    write!($w, "</{}>", stringify!($tag));
    write_html!($w, $($rest)*);
    }};
    }
    
    fn main() {
    use std::fmt::Write;
    let mut out = String::new();
    
    write_html!(&mut out,
    html[
    head[title["Macros guide"]]
    body[h1["Macros are the best!"]]
    ]);
    
    assert_eq!(out,
    "<html><head><title>Macros guide</title></head>\
     <body><h1>Macros are the best!</h1></body></html>");
    }

## 调试宏代码

为了看到宏扩展的结果，运行 rustc --pretty expanded。输出代表一个整体运行结果，所以你也可以把结果存入 rustc，它有时候会比原始的编译产生更好的错误信息。注意，如果多个同名的变量(但在不同的语法语境内)在相同的范围内起作用，--pretty expanded 的输出可能有不同的意义。在这种情况下，--pretty expanded，hygiene 会告诉你关于语法语境的情况。　　　　

rustc 提供了两种语法扩展以帮助宏调试。现在,他们是不稳定的。　　　　

1. log_syntax!(...) 将其参数打印到标准输出，在编译时，没有“扩展”。　　　
2. trace_macros!(true) 在每次宏扩展时产生编译器信息。在扩展结束时使用 trace_macros!(false) 。

## 语法要求

即使当 Rust 代码包含 un-expanded 宏时，它可以解析为一个完整的语法树。这个属性对编辑器和其他处理代码的工具是非常有用的。它也对 Rust 宏系统的设计产生一些后果。　　　　

一个后果是，解析一个宏调用时 Rust 必须确定宏是否代表

- 零个或多个项目，
- 零个或多个方法， 　　
- 一个表达式， 
- 一个语句，或　　
- 一个模式。

在代码块中的宏调用可以代表一些项目，或者一个表达式或语句。Rust 使用一个简单的规则来消除这个不确定性。宏调用的代表项目必须是

- 由花括号分隔开，例如 foo! { ... } ，或者
- 由一个分号终止，例如 foo!(...);

扩展前解析另一个后果是宏调用必须由有效 Rust 符号组成。此外，圆括号，方括号，花括号必须在一个宏调用中是平衡的。例如，foo!([)是禁止的。这允许 Rust 知道宏调用在哪里结束。　　　　

更正式地，宏调用体必须是一个“标记树”序列。一个标记树递归地定义为

- 一系列匹配的由 () ，[] ，或 {} 包围的标记树，或者
- 任何其它单个标记

在一个匹配器中，每个元变量都有一个“片段说明符”，来识别匹配哪些语法形式。

- ident：标识符。例如：`x; foo`。
- path：一个合格的名字。例如： `T::SpecialA`。
- expr： 一个表达式。例如：`2 + 2; if true then { 1 } else { 2 }; f(42)`。
- ty：一个类型。例如：`i32; Vec<(char, String)>; &T`。
- pat：一个模式。例如： `Some(t); (17, 'a'); _` 。
- stmt：单个语句。例如： `let x = 3`。
- block：一个括号分隔的语句序列。例如： `{ log(error, "hi"); return 12; }`。
- item：一个项目。例如： `fn foo() { }; struct Bar;` 。
- meta：一个 "元项目"， 在属性中建立的。 例如： `cfg(target_os = "windows")`。
- tt：一个单个标记树。

还有其他关于元变量后下一个标记的附加规则：

- 变量 expr 后必须加下面中的一个： =>  ,  ;
- 变量 ty 和 path 后必须加下面中的一个： => , : = > as
- 变量 pat 后必须加下面中的一个：=> , =
- 其它变量后可能要加其它符号。

这些规则提供一些在不破坏现有宏的情况下，Rust 语法发展的灵活性。

宏系统不处理解析的不明确性。例如，语法 `$($t:ty)* $e:expr` 总是无法解析，因为解析器将被迫选择解析 $t 和解析 $e。将调用语法改为在前面加一个独特的符号可以解决这个问题。在这种情况下，你可以编写 `$(T $t:ty)* E $e:exp`。

## 范围和宏导入/导出

宏在编译的早期阶段名称解析前被扩展。一个缺点是，相对于其他结构的语言，范围工作原理不同。　　　　

宏的定义和扩展都发生在一个 crate 资源的深度优先，词序遍历。所以一个定义在模块范围内的宏对在同一个模块内任何后续代码是可见的，其中包括任何后续的孩子 mod 项目的主体。

一个定义在单个 fn 的体内的宏，或其它不在模块范围的任何地方，只有在这个项目内是可见的。　　　　

如果一个模块有 macro\_use 属性，其宏在它的孩子模块的 mod 项目后的父模块中是可见的。如果父模块也有 macro\_use 属性，那么宏在父模块的 mod 项目后的祖父模块中也是可见的，等等。　　　　

macro_use属性也可以出现在 extern crate。在这种情况下它控制从 extern crate 加载哪些宏，如

    #[macro_use(foo, bar)]
    extern crate baz;

如果属性被简单定义如 #[macro\_use]，所有宏被加载。如果没有 #[macro\_use] 那么 宏就不能被加载。只有定义 #[macro_export] 属性的宏可能被加载。　　　　

为了加载没有连接到输出的 crate 的宏，使用 #[no_link]。　　　　

一个例子:

    macro_rules! m1 { () => (()) }
    
    // visible here: m1
    
    mod foo {
    // visible here: m1
    
    #[macro_export]
    macro_rules! m2 { () => (()) }
    
    // visible here: m1, m2
    }
    
    // visible here: m1
    
    macro_rules! m3 { () => (()) }
    
    // visible here: m1, m3
    
    #[macro_use]
    mod bar {
    // visible here: m1, m3
    
    macro_rules! m4 { () => (()) }
    
    // visible here: m1, m3, m4
    }
    
    // visible here: m1, m3, m4

当用 #[macro_use] extern crate 加载这个库时，只有 m2 将被导入。　　　　

Rust 参考有一个宏相关属性的列表。

## 变量 $crate

进一步困难发生在当一个宏在多个 crates 被使用。也就是 mylib 定义如下

    pub fn increment(x: u32) -> u32 {
    x + 1
    }
    
    #[macro_export]
    macro_rules! inc_a {
    ($x:expr) => ( ::increment($x) )
    }
    
    #[macro_export]
    macro_rules! inc_b {
    ($x:expr) => ( ::mylib::increment($x) )
    }

 inc\_a 只在 mylib 起作用，同时 inc\_b 只能在库外起作用。此外，如果用户在另一个名字下引入 mylib ，inc_b 将失去作用 。　　　　

Rust 没有针对 crate 参考的卫生系统，但它确实提供了一个解决这个问题的简单方法。在一个从一个名为foo的 crate 引入的宏，特殊宏变量 $crate 将扩展到 ::foo。相反，当一个宏被定义，然后在同一 crate 中被使用，$crate 就不会扩展。这意味着我们可以这样写

    #[macro_export]
    macro_rules! inc {
    ($x:expr) => ( $crate::increment($x) )
    }

来定义一个在库内库外的宏。函数名也扩展到 ::increment 或者 ::mylib::increment。

为了保持这个系统简单而正确，#[macro_use] extern crate ... 可能只出现在 crate 的根部，而不是 mod 内部。这将确保 $crate 是一个标识符。

## 深端

介绍性章节曾今提到过递归宏，但它没有给出完整的描述。递归宏是有用的另一个原因：每个递归调用给你匹配宏参数的另一个机会。　　　　

作为一个极端的例子，尽管不明智，在 Rust 的宏系统实现位循环标记自动机是可能的。

    macro_rules! bct {
    // cmd 0:  d ... => ...
    (0, $($ps:tt),* ; $_d:tt)
    => (bct!($($ps),*, 0 ; ));
    (0, $($ps:tt),* ; $_d:tt, $($ds:tt),*)
    => (bct!($($ps),*, 0 ; $($ds),*));
    
    // cmd 1p:  1 ... => 1 ... p
    (1, $p:tt, $($ps:tt),* ; 1)
    => (bct!($($ps),*, 1, $p ; 1, $p));
    (1, $p:tt, $($ps:tt),* ; 1, $($ds:tt),*)
    => (bct!($($ps),*, 1, $p ; 1, $($ds),*, $p));
    
    // cmd 1p:  0 ... => 0 ...
    (1, $p:tt, $($ps:tt),* ; $($ds:tt),*)
    => (bct!($($ps),*, 1, $p ; $($ds),*));
    
    // halt on empty data string
    ( $($ps:tt),* ; )
    => (());
    }

练习：使用宏来减少上述 bct! 宏的定义中的复制。

## 常见的宏

下面是一些你会在 Rust 代码中常见的宏。

### panic!

这个宏会导致当前线程的叛逆。你可以给它一个消息产生叛逆：

    panic!("oh no!");

####vec!

vec !宏在整本书被使用，所以你可能已经见过了。它毫不费力地创建 Vec<T>：

    let v = vec![1, 2, 3, 4, 5];

它还允许你用重复的值构建向量。例如，一百个 0：

    let v = vec![0; 100];

### assert! 和 assert_eq!

这两个宏用于测试。assert! 传入一个布尔值，assert_eq! 传入两个值并且比较它们。像下面这样：

    // A-ok!
    
    assert!(true);
    assert_eq!(5, 3 + 2);
    
    // nope :(
    
    assert!(5 < 3);
    assert_eq!(5, 3);
    

### try!

try! 用于错误处理。它传入可以返回 `Result<T, E>` 的参数，并给出 T 如果它是 Ok<T>，并且返回 Err(E) 。像这样：

    use std::fs::File;
    
    fn foo() -> std::io::Result<()> {
    let f = try!(File::create("foo.txt"));
    
    Ok(())
    }

这比这样做干净：

    use std::fs::File;
    
    fn foo() -> std::io::Result<()> {
    let f = File::create("foo.txt");
    
    let f = match f {
    Ok(t) => t,
    Err(e) => return Err(e),
    };
    
    Ok(())
    }
    
### unreachable!

当你认为一些代码不应该执行是使用这个宏：

    if false {
    unreachable!();
    }

有时，编译器可能会让你有一个不同的分支，你知道它永远不会运行。在这些情况下，使用这个宏，这样如果你错了，你会得到一个关于它的 panic!。

    let x: Option<i32> = None;
    
    match x {
    Some(_) => unreachable!(),
    None => println!("I know x is None!"),
    }

### unimplemented!

当你想让你的函数检查类型，不想担心写出函数的主体时 unimplemented! 宏可以被使用。这种情况的一个例子是你想要一次同时用多个所需的方法实现一个特征。把其它的定义为 unimplemented! 直到你准备写他们。

## 程序宏

如果 Rust 宏系统做不到你需要的，你可能想要编写一个编译器插件。相比 macro_rules !宏，这需要更多的工作，不太稳定的接口，bugs 可能更难追踪。作为交换你获得了在编译器内运行任意 Rust 代码的灵活性。这就是语法扩展插件有时被称为“程序宏”的原因。

在效率和可重用性方面， vec! 在 libcollections 的实际定义不同于这里介绍的。