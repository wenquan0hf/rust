#Hello, Cargo!

Cargo 是 Rustaceans 用来帮助管理他们的Rust项目的一个工具。Cargo目前处在 pre-1.0 状态，所以它仍然是一项正在进行中的项目。然而，它已经足够用于许多 Rust 的项目，所以我们就假设 Rust 项目从一开始就将使用 Cargo。

Cargo 管理三个方面的事情：构建代码，下载代码所需要的依赖，构建这些依赖项。前期阶段，你的程序没有任何的依赖，所以我们只使用其功能的第一部分。最终，我们会将添加更多依赖。从我们使用 Cargo 开始，添加依赖将会变得很容易。

如果你通过官方安装程序安装的 Rust，那么你也将拥有 Cargo。如果你用其他方法安装的 Rust，你可能希望查看 Cargo 的[自述](https://github.com/rust-lang/cargo#installing-cargo-from-nightlies)，通过特定的指令来安装它。

##转换成Cargo

让我们将Hello World 转换成 Cargo。

将我们的项目转换成 Cargo，我们需要做两件事情：创建一个 Cargo.toml 配置文件，把我们的源文件放在正确的地方。让我们先完成这部分：

	$ mkdir src
	$ mv main.rs src/main.rs

注意：因为我们创建的是一个可执行文件，所以我们使用的是 main.rs 文件。而如果我们想创建一个库文件，我们应该使用 lib.rs。自定义文件的位置入口点可以在下面描述的 TOML 文件中用 [[lib]] 或者 [[bin]] 关键字指定。

Cargo 期望你的源文件放在 src 目录中。这与顶层的其他东西隔离开来，比如 READMEs，许可证信息和其他任何与代码无关的东西。Cargo 可以帮助我们保持项目的漂亮整洁。万物各得其所。

接下来，编辑我们的配置文件：

	$ editor Cargo.toml

一定要确保这个名字是正确的：首字母 C 要大写！

把下面的内容输入到文件里面：
	
	[package]
	
	name = "hello_world"
	version = "0.0.1"
	authors = [ "Your name <you@example.com>" ]


这个文件是 TOML 格式的。让它向你来做个自我介绍吧：

	TOML 旨在成为一个最小的配置文件格式，由于明显的语义的使用，使其容易阅读。
    TOML 旨在明确的映射到一个哈希表。在各种各样的语言中，TOML应该易于解析成数据结构。

TOML与 INI 非常类似，但比其有一些额外的优点。

一旦你有了这个文件，我们应该准备构建了！试试这个：
	
	$ cargo build
	   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
	$ ./target/debug/hello_world
	Hello, world!

嘭！我们用 cargo build 构建我们的项目，并且用 ./target/debug/hello_world 运行它。我们可以用 cargo run 一步做到上面两件事：

	$ cargo run
	     Running `target/debug/hello_world`
	Hello, world!

请注意，这一次我们没有重建项目。Cargo 指出我们没有改变源文件，所以它只是运行的二进制文件。如果我们做了修改，我们会看到它的改变：

	$ cargo run
	   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
	     Running `target/debug/hello_world`
	Hello, world!

这没有简单的使用 rustc 来进行整个过程。想想在将来：当我们的项目变得更加复杂时，我们需要做更多的事情让所有的部分正确编译。如果利用 Cargo，随着我们的项目增长，我们仅仅用 cargo build 即可，它就会以正确的方式运行。

当你的项目终于准备发布了，你可以使用 cargo build --release 用最优化的方式编译你的项目。

你可能还会注意到，Cargo 已经创建了一个新的文件： Cargo.lock。
	
	[root]
	name = "hello_world"
	version = "0.0.1"


Cargo 用这个文件跟踪你应用程序中的依赖关系。现阶段，我们没有任何依赖，所以显得有点稀疏。你从不需要编辑该文件，让 Cargo 处理它就好了。

好了！我们已经成功地利用 Cargo 构建了 hello_world。尽管我们的程序很简单，但是我们开始使用的开拓未来 Rust 事业的真正工具了。你可以期待利用如下所示的做法，开始几乎所有的 Rust 项目：

	$ git clone someurl.com/foo
	$ cd foo
	$ cargo build

##新建一个项目

每次当你想开始一个新的项目时，你不必经历整个过程！Cargo 有能力做一个基本的项目目录，你可以在此基础上马上开始开发。

用 Cargo 开始一个新项目，使用命令 cargo new：

	$ cargo new hello_world --bin

我们传递了参数 --bin，因为我们正在创建一个二进制程序：如果我们创建的是一个库文件，就不要这个参数了。

让我们看看 Cargo 为我们生成了什么：

	$ cd hello_world
	$ tree .
	.
	├── Cargo.toml
	└── src
	    └── main.rs
	
	1 directory, 2 files

如果你没有 tree 命令，你可以从你的发行版的包管理器中获取。它不是必需的，但是它确实有用。

这是开始我们需要的全部。首先，让我们看看 Cargo.toml：

	[package]
	
	name = "hello_world"
	version = "0.0.1"
	authors = ["Your Name <you@example.com>"]

Cargo 基于你提供的参数作为合理的默认值填充该文件，你可用 git 获取全局配置。你可能会注意到，Cargo 也初始化 hello_world 目录为一个 git 仓库。

下面是 src/main.rs 里面的内容：
	
	fn main() {
	    println!("Hello, world!");
	}

Cargo 已经为我们生成了一个 “Hello World ！“，你可以开始编码了！Cargo 有自己的[指南](http://doc.crates.io/guide.html)，它涵盖 Cargo 许多更有深度的特性。

现在您已经了解了工具，让我们了解更多关于 Rust 语言本身的内容。这些基础知识将使你在剩下的时间里更好的理解 Rust。

你有两个选择：深入项目，进入章节”学习 Rust”，或者从基础开始，学习“语法和语义”章节。更有经验的系统程序员可能会更喜欢“学习 Rust”，而拥有动态语言背景的可能两者皆可。不同的人有不同的学习方式！选择适合你的。