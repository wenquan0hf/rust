#生存期  

这篇指南是 Rust 已经存在的三个所有权制度之一。这是 Rust 最独特和最令人信服的一个特点，Rust 开发人员应该相当熟悉。所有权即 Rust 如何实现其最大目标和内存安全。这里有几个不同的概念，每一个概念都有它自己的章节：  

- <a href="http://doc.rust-lang.org/stable/book/ownership.html">所有权</a>，即正在读的这篇文章。  

- <a href="http://doc.rust-lang.org/stable/book/references-and-borrowing.html">借用</a>，和与它们相关的功能‘引用’  

- 生存期，借用的先进理念 

这三篇文章相关且有序。如果你想完全的理解所有权制度，你将需要这三篇文章。    

#元
  
在我们了解细节之前，这里有关于所有权制度的两个重要说明需要知道。  

Rust 注重安全和速度。它通过许多‘零成本抽象’来完成这些目标，这意味着在 Rust 中，用尽可能少的抽象成本来保证它们正常工作。所有权制度是一个零成本抽象概念的一个主要例子。我们将在这篇指南中提到的所有分析都是在*编译时完成*的。你不用为了任何功能花费任何运行成本。  

然而，这一制度确实需要一定的成本：学习曲线。许多新用户使用我们喜欢称之为‘与借检查器的人斗争’，即 Rust 编译器拒绝编译那些作者认为有效的程序的 Rust 经验。这往往因为程序员对于所有权的怎样工作与 Rust 实现的规则不相符的心理模型而经常出现。你可能在第一次时会经历类似的事情。然而有个好消息：更有经验的 Rust 开发者报告称，一旦他们遵从所有权制度的规则工作一段时间后，他们会越来越少的与借检查器的行为斗争。  

学习了这些后，让我们来了解生存期。   

#生存期 

借出一个对于其他人已经拥有的资源的引用会很复杂。例如，假设这一系列的操作：  

- 我获得某种资源的一个句柄。  

- 我借给你对于这个资源的引用。  

- 我决定我使用完了这个资源，然后释放它，然而你仍然拥有这个引用。  
	
- 你决定使用资源。  

啊哦！你的引用指向一个无用的资源。当资源是内存时，这被称为悬挂指针或者‘释放后再利用’。  

要解决此类问题，我们必须确保在第三步后一定不会发生第四步。Rust 的所有权制度通过被称为生存期的一章来实现，生存期用来介绍一个引用的有效的作用域。  

当我们有一个函数将一个引用作为参数时，我们可以用隐式和显式两种方式来表示引用的生存期：

    // implicit
    fn foo(x: &i32) {
    }
    
    // explicit
    fn bar<'a>(x: &'a i32) {
    }

**'a** 读‘生存期为 a ’。从技术上讲，每个引用都有一些与之关联的生存期，但是编译器允许你在一般情况下忽略它们。但是在此之前，以下是我们分解显式例子的代码：  
    
    fn bar<'a>(...)

这一部分声明了我们的生存期。这说明 **bar** 有一个生存期 **'a**。如果我们有两个引用的参数，以下是相关代码：  
    
    fn bar<'a, 'b>(...)
    
然后在我们的参数列表中，我们使用我们已经命名的生存期。  

    ...(x: &'a i32)

如果我们想要一个 **&mut** 引用，我们可以书写以下代码：  

    ...(x: &'a mut i32) 

如果你将 **&mut i32** 和 **&'a mut i32** 比较，它们是相同的，只是在 **&** 和 **mut i32** 之间多了一个 **'a**。我们将 **&mut i32** 读作‘对于 i32 的一个可变引用’，将 **&'a mut i32**读作‘对于 **i32** 的一个生存期为**'a** 的一个可变引用’。  

当你操作<a href="http://doc.rust-lang.org/stable/book/structs.html">结构体</a>时，也需要显式的生存期：  

    struct Foo<'a> {
    x: &'a i32,
    }
    
    fn main() {
    let y = &5; // this is the same as `let _y = 5; let y = &_y;`
    let f = Foo { x: y };
    
    println!("{}", f.x);
    }

正如你所看到的的，**结构体**也可以有生存期。与函数相似的方式，
    
    struct Foo<'a> {

声明一个生存期，和以下代码

    x: &'a i32,

使用它。所以，为什么我们在这里需要一个生存期？我们需要确保对 **Foo** 的任何引用都不能比对它包含的 **i32** 的引用的寿命长。  

##作用域的考虑  

考虑生存期的一种方式是将一个引用的有效作用域可视化。例如：  

    fn main() {
    let y = &5; // -+ y goes into scope
    //  |
    // stuff//  |
    //  |
    }   // -+ y goes out of scope

在我们的 **Foo** 中添加:  

    struct Foo<'a> {
    x: &'a i32,
    }
    
    fn main() {
    let y = &5;   // -+ y goes into scope
    let f = Foo { x: y }; // -+ f goes into scope
    // stuff  //  |
      //  |
    } // -+ f and y go out of scope

我们的 **f** 在 **y** 的作用域内存活，所以一切正常。如果它不是呢？这个代码不会有效工作：  

    struct Foo<'a> {
    x: &'a i32,
    }
    
    fn main() {
    let x;// -+ x goes into scope
      //  |
    { //  |
    let y = &5;   // ---+ y goes into scope
    let f = Foo { x: y }; // ---+ f goes into scope
    x = &f.x; //  | | error here
    } // ---+ f and y go out of scope
      //  |
    println!("{}", x);//  |
    } // -+ x goes out of scope

正如你所看到的，**f** 和 **y** 的作用域比 **x** 的作用域要小。但是，当我们运行 **x = &f.x**  时，我们给了 **x** 一个可以超出其作用域范围的引用。  

命名生存期是给这些作用域命名的一种方式。给东西命名是能不能讨论它的第一步。  

##'static 

命名为 ‘static’ 的生存期是一个特殊的生存期。这标志着这种东西具有整个程序的生存期。很多的 Rust 程序员在处理字符串时会第一次遇到 **'static**：


    let x: &'static str = "Hello, world.";

字符串具有 **&'static str**  这种类型，是因为这种引用始终存在：它们融入到最终二进制的数据段中。另一个例子是全局变量：  

    static FOO: i32 = 5;
    let x: &'static i32 = &FOO;

以上代码是将 **i32** 加入到二进制文件的数据段中，其中 **x** 是它的一个引用。  

##生存期省略

Rust 在函数体中支持强大的局部类型推断，但是它在项目签名中禁止允许仅仅基于单独的项目签名中的类型推断。然而，对于人体工学推理来说被称为“生存期省略”的一个非常受限的二级推理算法，对于函数签名非常适用。它能推断仅仅基于签名组件本身而不是基于函数体，仅推断生存期参数，并且通过仅仅三个容易记住和明确的规则来实现，这使得生存期省略成为一个项目签名的缩写，而不是引用它之后隐藏包含完整的本地推理的实际类型。  

当我们谈到生存期省略时，我们使用*生存期输入*和*生存期输出* 这两个术语。*生存期输入*是与一个函数的一个参数结合的一个生存期，同时一个*生存期输出*是一个与函数的返回值相结合的一个生存期。例如，以下函数有一个生存期输入：  

    fn foo<'a>(bar: &'a str)

以下函数有一个生存期输出:  

    fn foo<'a>() -> &'a str

以下函数在两个位置都有一个生存期：  

    fn foo<'a>(bar: &'a str) -> &'a str

这里有三个规定：  

- 在函数参数中每个省略的生存期都成为一个独特的生存期参数。  

- 如果仅仅有一个输入生存期，省略或者不省略，在这个函数的返回值中，这个生存期被分配给所有的省略的生存期。  

- 如果这里有多个输入生存期，但是其中之一是 **&self** 或者 **&mut self**，这个生存期的 **self** 被分配给所有省略的生存期输出。  

另外，省略一个生存期输出是错误的。  

##例子

以下列举了生存期省略的函数的一些例子。我们已经将每个生存期省略的例子和它的扩展形式进行了配对。  

    fn print(s: &str); // elided
    fn print<'a>(s: &'a str); // expanded
    
    fn debug(lvl: u32, s: &str); // elided
    fn debug<'a>(lvl: u32, s: &'a str); // expanded
    
    // In the preceding example, `lvl` doesn’t need a lifetime because it’s not a
    // reference (`&`). Only things relating to references (such as a `struct`
    // which contains a reference) need lifetimes.
    
    fn substr(s: &str, until: u32) -> &str; // elided
    fn substr<'a>(s: &'a str, until: u32) -> &'a str; // expanded
    
    fn get_str() -> &str; // ILLEGAL, no inputs
    
    fn frob(s: &str, t: &str) -> &str; // ILLEGAL, two inputs
    fn frob<'a, 'b>(s: &'a str, t: &'b str) -> &str; // Expanded: Output lifetime is unclear
    
    fn get_mut(&mut self) -> &mut T; // elided
    fn get_mut<'a>(&'a mut self) -> &'a mut T; // expanded
    
    fn args<T:ToCStr>(&mut self, args: &[T]) -> &mut Command // elided
    fn args<'a, 'b, T:ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command // expanded
    
    fn new(buf: &mut [u8]) -> BufWriter; // elided
    fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a> // expanded
    

		
