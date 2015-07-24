## 文档

文档是软件项目的重要组成部分，而且它在 Rust 中是第一位的。让我们看看怎么使用 Rust 给出的工具来记录你的项目。

### 关于 rustdoc

Rust发行版包含一个工具 **rustdoc**，它可以生成一个文档。**rustdoc**通常也被 Cargo 通过 **cargo doc** 来使用。

文档可以通过两种方式生成：从源代码生成，或者从独立的 Markdown 文件生成。

### 源代码文档制作

记录一个 Rust 项目的主要方式是通过注释源代码。为此可以使用文档注释：


    /// Constructs a new `Rc<T>`.
    ///
    /// # Examples
    ///
    /// ```
    /// use std::rc::Rc;
    ///
    /// let five = Rc::new(5);
    /// ```
    pub fn new(value: T) -> Rc<T> {
    	// implementation goes here
    }

这段代码生成如下所示的文档。在这里只保留了它本来位置的常规的注释，而省去了实现。关于这个注释的第一件需要注意的事：它使用 `///`，而不是 `//`。即三重斜线表示文档的注释。

在 markdown 中书写文档的注释。

Rust 跟踪这些注释，并在生成文档时使用它们。当记录一些诸如枚举时，这是重要的：

    /// The `Option` type. See [the module level documentation](../) for more.
    enum Option<T> {
    	/// No value
		None,
    	/// Some value `T`
    	Some(T),
    }

上面的代码会正常执行，但是下面的代码不会正常执行：

    /// The `Option` type. See [the module level documentation](../) for more.
    enum Option<T> {
    	None, /// No value
    	Some(T), /// Some value `T`
    }

这时你会得到一个错误:

    hello.rs:4:1: 4:2 error: expected ident, found `}`
    hello.rs:4 }

这个 **unfortunate error** 是正确的：文档注释适用于紧随其后的内容，但是不能出现在语句的最后。

### 编写文档注释

不管怎样，让我们先详细了解每个部分的注释:

    /// Constructs a new `Rc<T>`.

第一行的文档注释应该是功能的一个简短的摘要。一个句子，只需要是最基本的，高水平的。

    ///
    /// Other details about constructing `Rc<T>`s, maybe describing complicated
    /// semantics, maybe additional options, all kinds of stuff.
    ///

我们最初的例子只是一行注释，但如果我们有更多的事情需要说明，我们可以在一个新的段落添加更多的解释。

#### 特殊部分

    /// # Examples

接下来是特殊部分。这些以一个头 # 表示。有三种常用的头文件。他们没有特殊的语法，现在只是有约定俗成的用法，

    /// # Panics

在Rust中，一个函数出现不可恢复的错误(即编程错误)通常由 panics 表示，它会至少结束整个当前线程。如果你的函数有这样一个重要的约定，它可以被 panics 检测/执行，记录就变得非常重要。

    /// # Failures

如果你的函数或方法返回一个**Result<T, E>**，这个结果描述了现在的状况，如果返回 **Err(E)**，表示现在状况还不是很坏。这和 **Panics** 相比就显得稍微不那么重要了，因为错误已经被编码，但它仍然不是一件特别坏的事。

    /// # Safety

如果你的功能是 **unsafe** 的，你应该向调用者解释哪些不变量是负责维护的。

    /// # Examples
    ///
    /// ```
    /// use std::rc::Rc;
    ///
    /// let five = Rc::new(5);
    /// ```
    
第三，Examples。包括一个或多个使用你的函数或方法的例子，你的用户会因为这些函数或方法而爱你。这些例子都包含了代码块注释，我们将讨论这部分：

    /// # Examples
    ///
    /// Simple `&str` patterns:
    ///
    /// ```
    /// let v: Vec<&str> = "Mary had a little lamb".split(' ').collect();
    /// assert_eq!(v, vec!["Mary", "had", "a", "little", "lamb"]);
    /// ```
    ///
    /// More complex patterns with a lambda:
    ///
    /// ```
    /// let v: Vec<&str> = "abc1def2ghi".split(|c: char| c.is_numeric()).collect();
    /// assert_eq!(v, vec!["abc", "def", "ghi"]);
    /// ```

让我们讨论一下这些代码块的细节。

#### 代码块说明

使用三重斜线对 Rust 代码进行注释：

    /// ```
    /// println!("Hello, world");
    /// ```

如果你想要某些代码不是 Rust 代码，您可以添加一个注解：

    /// ```c
    /// printf("Hello, world\n");
    /// ```

根据你选择的任何语言突出显示这些内容。如果你只想显示纯文本，只需要选择 **text** 就可以了。

选择正确的注释是很重要的，因为 **rustdoc** 以一个有趣的方式使用注释：实际上它可以用来测试你的例子，所以，他们离不开日期。如果你有一些C代码，但 rustdoc 认为它是 Rust，因为你没有使用注释，当试图生成文档 **rustdoc** 会产生异常。

### 文档测试

让我们讨论一下文档样例示例：

    /// ```
    /// println!("Hello, world");
    /// ```

你会注意到，你不需要一个 **fn main()** 或任何东西。**rustdoc** 会在你的代码中的正确位置自动添加一个 main()函数。例如：

    /// ```
    /// use std::rc::Rc;
    ///
    /// let five = Rc::new(5);
    /// ```

这是最终的测试：

    fn main() {
    	use std::rc::Rc;
    	let five = Rc::new(5);
    }

这是 rustdoc 使用后处理例子的完整算法：

- 任何主要 **#!(foo)** 属性是完好的 crate 属性。
- 一些常见的 **allow** 属性被插入，包括 **unused_variables** ，**unused_assignments**，**unused_mut**，**unused_attributes**，**dead_code**。小例子经常会引发这些麻烦。
- 如果样例不包含 **extern crate**，那么 **extern crate <mycrate>** 就会被插入。
- 最后，如果样例不包含 **fn main**，文本的其余部分会包装在fn main() { your_code }。

然而，有时这还是不够的。例如，所有这些代码示例中，我们用 `///` 来标注我们在说什么，原始文本：

    /// Some documentation.
    # fn foo() {}

输出看起来有些不同：

    /// Some documentation.

是的，这是正确的：你可以以 # 开始添加注释，输出时这些注释将被隐藏，编译代码时这些注释将会被使用。你可以利用这个优势。在这种情况下，文档注释需要适用于某种功能，所以如果我想给一个文档注释，我就需要添加一个小函数来定义它。同时，它只是为了满足编译器，所以隐藏它会使样例更加清晰。您可以使用这种方法来解释更详细的例子，同时仍然保留文档的可测试性。例如，这段代码：

    let x = 5;
    let y = 6;
    println!("{}", x + y);

下面呈现一个解释：

首先，我们将 x 设置为 5：
    
    let x = 5;

接下来，我们将 y 设置为 6：

    let y = 6;

最后，我们打印 x 和 y 的总和：
    
    println!("{}", x + y);

这是在原始文本中的相同的解释：
    
    First, we set x to five:
    
    let x = 5;
    # let y = 6;
    # println!("{}", x + y);

    Next, we set y to six:
    
    # let x = 5;
    let y = 6;
    # println!("{}", x + y);

    Finally, we print the sum of x and y:
    
    # let x = 5;
    # let y = 6;
    println!("{}", x + y);

通过重复例子的所有部分，您可以确保您的例子仍然可以编译，而只显示跟你的解释相关的那部分。

### 记录宏

这里有一个宏的例子:

    /// Panic with a given message unless an expression evaluates to true.
    ///
    /// # Examples
    ///
    /// ```
    /// # #[macro_use] extern crate foo;
    /// # fn main() {
    /// panic_unless!(1 + 1 == 2, “Math is broken.”);
    /// # }
    /// ```
    ///
    /// ```should_panic
    /// # #[macro_use] extern crate foo;
    /// # fn main() {
    /// panic_unless!(true == false, “I’m broken.”);
    /// # }
    /// ```
    #[macro_export]
    macro_rules! panic_unless {
    	($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
    }

你需要注意三件事：添加自己的 **extern crate** 行，这样我们就可以添加 **#[macro_use]** 属性。第二，我们需要添加自己的 **main()** 函数。最后，明智地使用 # 注释掉这两个东西，确保他们不会显示在输出中。

### 运行文档测试

运行测试

    $ rustdoc --test path/to/my/crate/root.rs
    # or
    $ cargo test

**cargo test** 也可以对嵌套文档进行测试。然而，**cargo test**只能测试函数库而不能测试二进制工具。这是由于 **rustdoc** 的工作方式：链接库进行测试，但如果是二进制的，就没有什么可链接的。

当测试你的代码时，这还有几个有用的注释，可以帮助 rustdoc 做正确的事情：

    /// ```ignore
    /// fn foo() {
    /// ```

**ignore** 指令告诉 Rust 忽略代码。这是最通用的。相反，如果它不是代码，可以考虑使用 **text** 注释，或者使用 **#**s 得到一个工作示例，这个工作示例只显示你关心的部分。

    /// ```should_panic
    /// assert!(false);
    /// ```

**should_panic** 告诉 **rustdoc** 应该正确地编译代码，但测试实例实际上并没有通过。

    /// ```no_run
    /// loop {
    /// println!("Hello, world");
    /// }
    /// ```

**no_run** 属性将编译代码，但不运行它。这是很重要的例子，例如 “Here's how to start up a network service,” 这表示您想要确保编译，但可能以无限循环的模式运行!

### 记录模块

Rust有另外一种文档评论 `//!`。这个评论不记录下一个项目，而是记录封闭的项目。换句话说:

    mod foo {
    	//! This is documentation for the `foo` module.
    	//!
    	//! # Examples
    
    	// ...
    }

`//!` 最常见的用途：用于模块文档。如果你在 **foo.rs** 之内有一个模块，你经会经常打开它的代码，并且看到这个：

    //! A module for using `foo`s.
    //!
    //! The `foo` module contains a lot of useful functionality blah blah blah

### 文档注释风格

查看 [RFC 505](https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md)，得到完整规范的风格和格式文档。

### 其他文档

所有这一切行为都工作于 non-Rust 源文件。因为评论都写在 Markdown 中，他们经常都是 **.md** 文件。

当你在 Markdown 文件中编写文档时，不需要使用评论给文档加前缀。例如：

    /// # Examples
    ///
    /// ```
    /// use std::rc::Rc;
    ///
    /// let five = Rc::new(5);
    /// ```

只需要

    # Examples
    
    ```
    use std::rc::Rc;
    
    let five = Rc::new(5);
    ```

这些代码存在于 Markdown 文件中。不过有一个问题：Markdown 文件需要一个这样的标题：

    % The title
    
    This is the example documentation.
    
注意：加 **%** 的行需要是文件的第一行。

### 文档属性

在更深的层面，文档注释是文档属性的装饰：

    /// this
    
    #[doc="this"]

如下是相同的：

    //! this
    
    #![doc="/// this"]

你不会经常看到这个属性用于编写文档，但当改变一些选项，或者当编写一个宏是，它可能是有用的。

### 双出口

**rustdoc** 将显示一个文档为了在两个位置得到一个公共再出口:

    extern crate foo;
    
    pub use foo::bar;

因为 bar 且在文档内部因为 crate foo 而创建文档。它将在两个地方使用相同的文档。

这种行为可以被 **no_inline** 禁止：

    extern crate foo;
    
    #[doc(no_inline)]
    pub use foo::bar;

### 控制 HTML

你可以通过 #!(doc) 的属性版本控制 rustdoc 产生 HTML 的若干部分：

    #![doc(html_logo_url = "http://www.rust-lang.org/logos/rust-logo-128x128-blk-v2.png",
       html_favicon_url = "http://www.rust-lang.org/favicon.ico",
       html_root_url = "http://doc.rust-lang.org/")];

这是一个不同选项的集合，包括标志，标识，和一个根 URL。


### 生成选项

**rustdoc** 还包含一些其他命令行选项，为了进一步用户化：

- **html-in-header FILE**：在 `<head>…</head>` 的结尾部分，包括文件的内容。
- **html-before-content FILE**：在 `<body>` 之后，呈现的内容之前(包括搜索栏)，包括文件的内容。
- **html-after-content FILE**：在所有呈现内容之后，包括文件的内容。

### 安全事项

Markdown 中放置的文档注释，没有被处理成最终的网页。小心文字的 HTML：
    
    /// <script>alert(document.cookie)</script>