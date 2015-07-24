# 变量绑定 

几乎所有非 “Hello World” Rust 程序使用变量绑定。如下的形式：

```
 fn main() {
	let x = 5;
 }
```

在每个例子中都写 fn main(){ 显得有点冗余，所以在以后我们会省略掉它。如果你一直跟着教程学习，确保修改你的 mian() 函数，而不是丢掉了它。否则，你的程序编译会得到一个错误。

在许多语言中，这被称为一个变量，但 Rust 中的变量绑定有一些小窍门需要注意。例如让表达式的左边是一个 “pattern”，而不仅仅是一个变量名。这意味着我们可以这样做:

```
 let (x, y) = (1, 2);
```

这个表达式求值后，x 将会被赋值为 1，y 将会被赋值为 2。[Pattern](https://doc.rust-lang.org/stable/book/patterns.html) 是非常的强大，在本教程中有专门的章节讲解。我们暂时不需要这些功能，所以我们就把暂时不关注这个，继续学习本节的知识。

Rust 是一种静态类型语言，这意味着我们要提前指定类型，并且在编译时期被检查。那么，为什么我们的第一个例子能够被编译？Rust 有个称为“类型推断”的机制。如果它能推断出变量是什么类型，Rust 就不需要指定实际类型。

如果想要我们可以添加它的类型。在冒号(:)之后输入类型：

```
 let x: i32 = 5;
```

如果我要求你将上面的大声读给班上的其他同学，你会说 “x 被绑定为 i32 类型，并且它的值为5。”

在这种情况下，我们选择将 x 表示为一个 32 位带符号整数。Rust 有许多不同的基本的整数类型。以 i 开头的表示有符号整型，u 开头的表示无符号整型。可能的整数尺寸是 8、16、32、64 位。

在以后的例子中，我们可能会在注释中说明其类型。以后的例子会看起来像这样:

```
 fn main() {
	let x = 5; // x: i32
 }
```

注意使用 let 的语法和注释表示的是相似的。和让您所使用的语法。Rust 中包含上面的注释方式，但它不是通用的方式，因此我们会偶尔的使用上面的注释方式来帮助你理解变量表示 Rust 中的实际类型。

默认情况下，绑定是不可变的。这段代码将不会编译通过:

```
 let x = 5;
 x = 10;
```

它将会输出如下的错误：

```
error: re-assignment of immutable variable `x`
     x = 10;
     ^~~~~~~
```

如果你想绑定是可变的，您可以使用 mut 关键字:

```
 let mut x = 5; // mut x:i32
 x = 10;
```

没有原因说明默认绑定是不可变的，但我们可以通过考虑 Rust 关注的一个要点：安全性。如果你忘了什么 mut，编译器会捕捉到它，让你知道你修改了某些值但是你可能没有声明该值为 mut。如果绑定默认情况下是可变的，那么编译器将无法告诉你这一点。如果你的确想修改变量值，那么解决方案很简单:添加 mut。

还有其他理由避免出现可变状态，但他们超出本指南的范围。一般来说，通常可以避免显式变化，那也是 Rust 所推荐的。也就是说，有时，修改变量是你需要的，但是它不是禁止的。

让我们回到绑定。Rust 变量绑定与其他语言有一个方面是不同：在你能够使用绑定变量之前你要初始化该变量。

让我们来自己动手试一试。修改 src/main.rc 文件，使其中的内容像下面的一样：

```
 fn main() {
 	let x: i32;
	
	println!("Hello world!");
 }
```

你可以在命令行中使用 cargo build 命令对该文件进行编译 。尽管你会得到一个警告，但是仍然会打印 ”Hello, world！“：

```
 Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:2:9: 2:10 warning: unused variable: `x`, #[warn(unused_variable)]
   on by default
src/main.rs:2     let x: i32;
                      ^
```

Rust 警告我们从来没有使用这个变量绑定，但是因为我们从来没有使用它，没有破坏性，没有恶意。然后，如果我们试图使用变量 x 事情就会发生变化。让我们来试一下。改变你的程序为如下的样子:

```
 fn main() {
	let x: i32;
 	
	println!("The value of x is:{}", x);
 }
```

尝试编译上面的代码。将会看到输出一个错误：

```
$ cargo build
   Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:4:39: 4:40 error: use of possibly uninitialized variable: `x`
src/main.rs:4     println!("The value of x is: {}", x);
                                                    ^
note: in expansion of format_args!
<std macros>:2:23: 2:77 note: expansion site
<std macros>:1:1: 3:2 note: in expansion of println!
src/main.rs:4:5: 4:42 note: expansion site
error: aborting due to previous error
Could not compile `hello_world`.
```

Rust 不会让我们使用未初始化的变量。接下来，让我们谈谈添加到 println！ 中的东西。

如果在你打印的字符串语句中包含花括号({})，Rust 将这个视为请求插入某种值。字符串插入值是一个计算机科学名词，它的意思是“粘在字符串的中间。“我们添加一个逗号，然后是 x，表明我们希望插入的是 x 的值。如果传递给函数和宏的参数不止一个，我们用逗号对参数进行分隔。

当你仅仅只是使用花括号，Rust 通过检查它的类型将尝试以一种有意义的方式显示它的值。如果你想用更详细的方式指定打印格式，请查看[多个可选参数可用](https://doc.rust-lang.org/stable/std/fmt/)。此时，我们只关注默认值：整数打印起来并不是很复杂。
