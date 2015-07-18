##箱和模块

当一个项目开始变大时, 我们通常认为的良好的软件工程实践是把它分割成小块,然后把它们组合在一起。同样重要的是有一个定义良好的接口,这样你的一些功能可以是私人的,另外一些可以是公开的。为了促进这些事情,Rust 使用了模块系统。

###基本术语:箱和模块

关与模块系统，Rust 有两个不同的术语:“箱” 和“模块”。 在其他语言里箱的代名词是 “库” 或 “包”。因此 “Cargo” 就是 Rust语言的的包管理工具:你可以用Cargo 装载你的箱转运给其他程序。根据不同的项目，箱可以产生一个可执行文件或库。

每个箱有一个包含箱代码的隐式根模块。然后,您可以定义一个根模块下的子树模块。
模块允许你分区箱内箱外代码。

作为一个例子,让我们做一个短语箱,它将在不同的语言中给我们不同的词语。为简单起见,我们将使用 “问候” 和 “告别” 两种类型的短语,并使用英语和日语 (日本语) 两种语言。我们将使用下面这个模块布局:

![image](http://img-storage.qiniudn.com/15-7-17/82526530.jpg)

     
                     

      
在这个例子中,短语是我们箱的名字。其余都是模块。你可以看到,他们形成一个树,分支从箱根发出,根指的是树的根:短语本身。

现在我们有一个计划,让我们来在代码中定义这些模块。首先,用 Cargo 生成一个新的箱:
    
    $ cargo new phrases
    $ cd phrases

如果你记得以前所讲的,这将为我们生成一个简单的项目:
    
    $ tree .
    .
    ├── Cargo.toml
    └── src
    └── lib.rs
    
    1 directory, 2 files

src/lib.rs 是我们箱根,对应于我们在上图中的短语。

###定义模块

我们使用 mod 关键字来定义我们的每个模块。让我们使我们的 src/lib.rs，看起来就像这样:

    mod english {
    mod greetings {
    }
    
    mod farewells {
    }
    }
    
    mod japanese {
    mod greetings {
    }
    
    mod farewells {
    }
    }
    
在 mod 关键字后,我们给出模块的名称。模块名称遵守 Rus t规定的标识符命名规则:lower_snake_case。每个模块的内容在花括号 ({ }) 里面。

在一个给定的模式下,您可以声明 sub-mods。我们可以用双冒号 (::) 符号引用子模块:我们的四个嵌套模块是 english::greetings, english::farewells, japanese::greetings, 还有 japanese::farewells。因为这些子模块是在他们父模块命名空间命名的,名字不冲突: english::greetings 和 japanese::greetings 是不同的,尽管他们的名字都是问候。

因为这个箱子没有 main() 函数,并且被称为 lib.rs, Cargo 将把这个箱建成一个库:

    $ cargo build
       Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
    $ ls target/debug
    build  deps  examples  libphrases-a7448e02a0468eaa.rlib  native

libphrase-hash.rlib 是编译后的箱。在我们知道如何在另一个箱里面使用这个箱之前,让我们把它分成多个文件。

###多个文件箱

如果每个箱只是一个文件,那么这些文件会很大。我们常常很容易将箱分成多个文件,并且 Rust 从两个方面来支持这样做。

不是像下面这样声明一个模块:

    mod english {
    // contents of our module go here
    }
    ⇱

相反我们可以这样声明我们的模块:

    mod english;

如果我们这样做,Rust 将期望找到一个 english.rs 文件, 或者是含有我们的模块的内容的english/mod.rs 文件。

注意,在这些文件中,您不需要 re-declare 模块:这些已经由最初的模块的声明了。

使用这两种技巧,我们可以把箱子拆分成两个目录和七个文件:

![image](http://img-storage.qiniudn.com/15-7-17/63405766.jpg)



src/lib.rs 是我们箱根,看起来像这样:

    mod english;
    mod japanese;

这两个声明告诉 Rust 根据我们的偏好去寻找 src/english.rs 和 src/japanese.rs, 或者 src/english/mod.rs 和 src/japanese/mod.rs。在这种情况下,由于我们的模块有子模块,我们就选择第二个。src/english/mod.rs 和 src/japanese/mod.rs 看起来都像这样:

    mod greetings;
    mod farewells; 
    
再一次,这些声明告诉 Rust 去寻找 src/english/greetings.rs 和 src/japanese/greetings.rs 或者 src/english/farewells/mod.rs 和 src/japanese/farewells/mod.rs。因为这些子模块没有自己的子模块,我们选择让他们 src/english/greetings.rs 和 src/japanese/farewells.rs。唷! !

src/english/greetings.rs 和 src/japanese/farewells.rs 的内容在此时都是空的。让我们添加一些函数。

把下面这些放到 src/english/greetings.rs 里面:

    fn hello() -> String {
    "Hello!".to_string()
    }
    把下面这些放到src/english/farewells.rs:
    fn goodbye() -> String {
    "Goodbye.".to_string()
    }
    src/japanese/greetings.rs:
    fn hello() -> String {
    "こんにちは".to_string()
    }
    
当然,你可以从这个网页复制和粘贴这些或者自己敲一些其他的东西。你用 “konnichiwa” 还是其他的什么学习模块系统实际上并不重要。

把下面这些放到 src /日本/ farewells.rs:
    
    fn goodbye() -> String {
    "さようなら".to_string()
    }
    
 (如果你好奇的话，可以告诉你这是 “Sayōnara”。)

现在,我们的箱具有一些功能,让我们试着从另一个箱使用这些功能。

###导入外部箱

我们有一个库箱。让我们做一个可执行的箱,这个箱导入和并且使用我们的库。

生成一个 src/main.rs 并且把下面这些代码输进去(此时还不会完全编译):


    extern crate phrases;
    
    fn main() {
    println!("Hello in English: {}", phrases::english::greetings::hello());
    println!("Goodbye in English: {}", phrases::english::farewells::goodbye());
    
    println!("Hello in Japanese: {}", phrases::japanese::greetings::hello());
    println!("Goodbye in Japanese: {}", phrases::japanese::farewells::goodbye());
    }

外面的箱声明告诉,我们需要编译和链接短语箱。我们可以使用短语“模块。如前所述,您可以使用双冒号来引用子模块的内部功能。

另外, Cargo 假设 src/main.rs 是一个二进制箱的根箱，而不是一个箱库。我们的包现在有两个箱: src/lib.rs 以及 src/main.rs。对可执行文件箱来说，这种模式是很常见的:大多数功能都是在库箱里面,并且可执行箱将使用这个库。在这种方式下,其他程序也可以使用库箱,这也是一个不错的关注点分离方法。

然而这并不管用。我们得到了类似下面四个的错误:

    $ cargo build
       Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
    src/main.rs:4:38: 4:72 error: function `hello` is private
    src/main.rs:4 println!("Hello in English: {}", phrases::english::greetings::hello());
       ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    note: in expansion of format_args!
    <std macros>:2:25: 2:58 note: expansion site
    <std macros>:1:1: 2:62 note: in expansion of print!
    <std macros>:3:1: 3:54 note: expansion site
    <std macros>:1:1: 3:58 note: in expansion of println!
    phrases/src/main.rs:4:5: 4:76 note: expansion site

默认情况下,在 Rus t语言里面一切都是非公开的。让我们从更深的层次来谈一下。

###导出一个公共接口

在默认情况下，Rust 可以精确地控制你的接口的哪些方面是公开的,哪些方面是非公开的。要把某些事物公开,你需要使用使用 pub 关键字。让我们首先关注 english 模块,然后让我们减小我们的 src/main.rs 到下面这样:
    
    extern crate phrases;
    
    fn main() {
    println!("Hello in English: {}", phrases::english::greetings::hello());
    println!("Goodbye in English: {}", phrases::english::farewells::goodbye());
    }
    
在我们的 src/english/mod.rs 里面，让我们添加 pub 到英语模块声明里面:
    pub mod english;
    mod japanese;


并且在我们的 src/english/mod.rs 里面,我们写两个 pub 语句:

    pub mod greetings;
    pub mod farewells;

在我们的 src/english/greetings.rs 里面,我们添加 pub 到 fn 的声明里面:

pub fn hello() -> String {
"Hello!".to_string()
}

也在 src/english/farewells.rs 里面这样做:
    
    pub fn goodbye() -> String {
    "Goodbye.".to_string()
    }
    
现在,虽然有警告告诉我们不能使用带有日语的函数，我们的箱依然进行编译:
    
    $ cargo run
       Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
    src/japanese/greetings.rs:1:1: 3:2 warning: function is never used: `hello`, #[warn(dead_code)] on by default
    src/japanese/greetings.rs:1 fn hello() -> String {
    src/japanese/greetings.rs:2 "こんにちは".to_string()
    src/japanese/greetings.rs:3 }
    src/japanese/farewells.rs:1:1: 3:2 warning: function is never used: `goodbye`, #[warn(dead_code)] on by default
    src/japanese/farewells.rs:1 fn goodbye() -> String {
    src/japanese/farewells.rs:2 "さようなら".to_string()
    src/japanese/farewells.rs:3 }
     Running `target/debug/phrases`
    Hello in English: Hello!
    Goodbye in English: Goodbye.

现在,我们的函数是公开的,我们可以使用它们。太棒了!然而,输入phrases::english::greetings::hello()太长而且重复。Rust 还有另一个关键字可以导入名称到当前的范围,这样你可以用更短的名字来引用他们。让我们谈谈 use。

###用use导入模块

Rust 有一个 us e关键字,它允许我们将名称导入本地范围。让我们改变我们的 src/main.rs 成下面这样:

    extern crate phrases;
    
    use phrases::english::greetings;
    use phrases::english::farewells;
    
    fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());
    }
    
两个 use 行将每个模块导入到本地范围,所以我们可以用更短的名称来调用函数。按照惯例,在导入功能时,通常认为最好的做法是导入模块而不是直接导入函数。换句话说,你可以这样做:

    extern crate phrases;
    
    use phrases::english::greetings::hello;
    use phrases::english::farewells::goodbye;
    
    fn main() {
    println!("Hello in English: {}", hello());
    println!("Goodbye in English: {}", goodbye());
    }
    
但它不是惯用的方法。这是更有可能引入命名冲突。在我们的短程序里面,这不是一个大问题,但是当它在大型程序里面就变成一个问题了。如果我们有相互矛盾的名字,Rust 会给出一个编译错误。例如,如果我们用 public 修饰日语函数,并试图做到这一点:

    extern crate phrases;
    
    use phrases::english::greetings::hello;
    use phrases::japanese::greetings::hello;
    
    fn main() {
    println!("Hello in English: {}", hello());
    println!("Hello in Japanese: {}", hello());
    }
    
Rust 将给我们一个编译时的错误:

    Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
    src/main.rs:4:5: 4:40 error: a value named `hello` has already been imported in this module [E0252]
    src/main.rs:4 use phrases::japanese::greetings::hello;
      ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    error: aborting due to previous error
    Could not compile `phrases`.

如果我们从相同的模块导入多个名称,我们不需要输入两次。不必像下面这样:

    use phrases::english::greetings;
    use phrases::english::farewells;

我们可以使用这个快捷键:

    use phrases::english::{greetings, farewells};

###用pub use重新导出

你不要只是使用 use 关键字来缩短标识符。您还可以在你的箱里使用它去再次导入一个在另一个模块里的函数。这允许您呈现一个外部接口,并且这个接口可以并不直接映射到您的内部代码组织。

让我们来看一个例子。修改您的 src/main.rs 像下面这样:

    extern crate phrases;
    
    use phrases::english::{greetings,farewells};
    use phrases::japanese;
    
    fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());
    
    println!("Hello in Japanese: {}", japanese::hello());
    println!("Goodbye in Japanese: {}", japanese::goodbye());
    }
    
然后,修改您的 src/lib.rs 使日语 mod 公开:
    
    pub mod english;
    pub mod japanese;
    
接下来,公开这两个函数,首先在 src/japanese/greetings.rs 里面:

    pub fn hello() -> String {
    "こんにちは".to_string()
    }

然后在 src/japanese/farewells.rs 里面:
    
    pub fn goodbye() -> String {
    "さようなら".to_string()
    }
    

最后,修改您的 src/japanese/mod.rs 成下面这样:

    pub use self::greetings::hello;
    pub use self::farewells::goodbye;
    
    mod greetings;
    mod farewells;
    
在我们模块的层次结构的这一部分，pub use 将在这个范围内声明函数。因为我们在日语模块里面已经这样做了，我们现在有一个 phrases::japanese::hello() 函数 和 一个 phrases::japanese::goodbye() 函数,虽然他们的代码存在于 phrases::japanese::greetings::hello() 和 phrases::japanese::farewells::goodbye()。我们内部组织不能定义我们的外部接口。

在这里，每个我们想纳入日语范围的函数都有一个 pub use。我们也可以使用通配符语法去吧 greating 里面的所有东西列入到当前范围: pub use self::greetings::*。

那 sel f呢?默认情况下,从你的箱根开始，use 的声明都是绝对路径,。相反，self 使这条路是一条相对于当前的层次结构的相对路径。还有一个 use 的特殊形式:您可以使用 super:: 达到你所在的树的当前层次的上一层。从许多 shell 的当前目录和父目录所显示的来看，有些人经常认为 self是.而 super是. .。
在 Use 的外部,路径是相对的: 相对于我们所处的位置，foo::bar() 指向的是一个 foo内部的函数。如果那是一个 :: 前缀,就像 ::foo:bar()里面的,那么它指的是一个不同的 foo,是一条从你的箱根开始的绝对路径。

同时,注意,我们在声明 mod 之前就 pub use 了。Rust 语言要求首先进行 use 的声明。

这将构建并运行下面的代码:

    $ cargo run
       Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
     Running `target/debug/phrases`
    Hello in English: Hello!
    Goodbye in English: Goodbye.
    Hello in Japanese: こんにちは
    Goodbye in Japanese: さようなら
    
    