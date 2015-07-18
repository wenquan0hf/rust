#猜谜游戏

我们的第一个项目，将实现一个典型的初学者编程的问题：猜谜游戏。下面介绍下它是如何工作的：我们的程序将生成一个从一到一百的随机整数。然后它会提示我们输入一个猜测值。依据我们的输入的猜测值，它会告诉我们猜测值是否过低或者过高。一旦我们猜正确，它将祝贺我们。听起来不错吧？

##设置

进入你的项目目录，让我们建立一个新项目。还记得我们必须为  hello_world 创建目录结构和 Cargo.toml 吗？Cargo 有一个命令能为我们做这些事。让我们来试一试：

	$ cd ~/projects
	$ cargo new guessing_game --bin
	$ cd guessing_game

我们向命令 cargo new 传递了我们项目的名称，然后添加了标记 --bin ，这是因为我们正在创建一个二进制文件，而不是一个库文件。

查看生成的 Cargo.toml：

	[package]
	
	name = "guessing_game"
	version = "0.1.0"
	authors = ["Your Name <you@example.com>"]

Cargo 从你的环境中得到这些信息。如果它是不正确的，进入文件并且改正过来。

最后，Cargo 为我们生成一个“Hello，world!”。查看下 src/main.rs：

	fnmain() {
	    println！("Hello， world！")
	}

让我们尝试编译下 Cargo 给我们的东西：
	
	$ cargo build
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)

太好了！再次打开你的 src/main.rs。我们将在这个文件中编写所有的代码。

在我们继续之前，让我教你一个 Cargo 的命令： run.cargo run 是有点像 cargo build 的命令，但是它可以在生成的可执行文件后运行此文件。试一试：

	$ cargo run
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	     Running `target/debug/guessing_game`
	Hello, world!

太棒了！当你需要在一个项目中快速迭代时，这个 run 命令可以派上用场。我们游戏就是这样的一个项目，在下一次迭代之前，我们需要快速测试每个迭代。

#处理一个猜测值

我们开始做吧！对于我们的猜谜游戏，我们需要做的第一件事就是允许玩家输入一个猜测值。将下面的内容输入到你的 src/main.rs：

	use std::io;
	
	fn main() {
	    println!("Guess the number!");
	
	    println!("Please input your guess.");
	
	    let mut guess = String::new();
	
	    io::stdin().read_line(&mut guess)
	        .ok()
	        .expect("Failed to read line");
	
	    println!("You guessed: {}", guess);
	}

这里面有很多代码啊！让我们一点一点地输入它。

	use std::io;

我们需要获取用户的输入，然后将打印结果作为输出。因此，我们需要标准库中的 io 库。Rust 利用“[序部](http://doc.rust-lang.org/stable/std/prelude/)”只要引入少量的东西到每一个项目，。如果在序部中没有的话，那么你必须直接 use 它。

	fn main() {

正如之前你所见过的， main() 函数是程序的入口点。这个 fn 语法声明了一个新函数， ()表明没有参数， {表示函数的主体开始。因为我们不包含一个返回类型，它假定是 ()，一个空元组。
	
	   println!("Guess the number!");
	
	   println!("Please input your guess.");

我们之前学习到的 println！() 是一个打印字符串到屏幕上的宏。

	let mut guess = String::new();


现在越来越有趣了！有很多东西在这个小行上。首先要注意的是，这是一个let 语句，是用来创建变量绑定的。他们以这种形式存在：

	let foo = bar;

这将创建一个名为 foo 的新绑定，并将其绑定到值 bar 上。在许多语言中，这被称为一个“变量”，但是 Rust的变量绑定有其成熟的方法。

例如，他们在默认情况下是不可变的。这就是我们的例子中使用 mut 的原因：它使一个绑定是可变，而不是不可改变的。 let 的左边不能添加名称，实际上，它可接受一个“模式”。之后，我们将更多的使用模式。现在它很容易使用：

	let foo = 5; // 不可变的.
	let mut bar = 5; // 可变的

哦， // 之后将开始一个注释，直到行的结束。Rust 编译时，将会忽略所有的注释。

所以现在我们知道了 let mut guess 将引入一个名为 gusss 可变的绑定，但我们必须看 = 另一边的绑定： String：：new()。
 
String是一个字符串类型，由标准库提供的。 String 是一个可增长，utf-8 编码的文本。

：：new() 语法使用 ：：因为这是一个特定类型的关联函数。也就是说，它与 String 本身相关，而不是一个特定的实例化的 String。一些语言称之为“静态方法”。

这个函数命名为 new()，因为它创建了一个新的，空的 String。你会在许多类型上发现 new()函数，因为它是创建某类型新值的一个普通的名称。

让我们继续往下看：

    io::stdin().read_line(&mut guess)
        .ok()
        .expect("Failed to read line");

这又有很多代码！让我们一点一点的看。第一行包含两部分。这是第一部分：

	io：：stdin()

还记得我们如何在程序的第一行使用 use std：：io 吗?我们现在调用一个与之相关的函数。如果我们不用 use std：：io，我们可以这样写这一行 std：：io：：stdin()。

这个特别的函数返回一个句柄到终端的标准输入。更具体地说，返回一个 std：：io：：Stdin。

下一部分将使用这个句柄获取来自用户的输入：

	.read_line(&mut guess)

在这里，我们在句柄上调用 read\_line() 方法。方法就像联合的方法，但其只能在一个特定的实例类型中使用，而不是在类型本身。我们还传递了一个参数到 read\_line()： &mut guess。

还记得我们上面是怎么绑定 guess 的吗?我们认为它是可变的。然而， read\_line 不是将一个 String 作为一个参数：它用 &mut String 做参数。Rust 有一个称为“引用”的特性，对一个数据你可以有多个引用，这样可以减少复制。引用是一个复杂的特性。如何安全和容易的使用引用是 Rust 的一个主要卖点。现在我们不需要知道很多这些细节来完成我们的项目。现在我们所需要知道的是像 let 绑定，引用是默认是不可变的之类的事情。因此，我们需要写代码 &mut guess，而不是 &guess。

为什么 read\_line() 采用一个可变的字符串引用做参数? 它的工作就是获取用户键入到标准输入的内容，并将转换成一个字符串。因此，以该字符串作为参数，因为要添加输入，所以它需要是可变的。

但是我们不确定这一行代码会执行正确。虽然它仅是一行文本，但是它是第一部分的单一逻辑行代码：

	    .ok()
        .expect("Failed to read line");

当你用类似 .foo() 的语法调用一个方法时，你可以用换行和空格来写开启新的一行代码。这能帮助你分开很长的代码行。我们之前的代码是这样写的：

	io::stdin().read_line(&mut guess).ok().expect("failed to read line");

但是这样的代码阅读性不好，所以我们将其分隔开。三个方法调用分成三行来写。我们已经讲过 read\_line() 了，那么 ok() 和 expect() 是什么呢?嗯，我们之前已经提到了， read\_line() 读取了用户输入到 &mut String 中的内容。而且它也返回了一个值：在上面的例子中，返回值是 io：：Result。Rust 在其标准库中有很多类型命名为 Result：一个通用的 Result，然后用子库指定具体的版本，比如 io：：Result。

使用这些 Result 类型的目的是编码错误处理信息。Result 类型的值，类似于任何类型，其有定义的方法。在上面的例子中，io：：Result 就有一个 ok() 方法。这个方法表明，“我们想假定这个值是一个正确值。如果不是，那么就抛出错误的信息。”那么为什么要抛出它？对于一个基本的程序，我们只是想打印出一个通用的错误信息。任何这样的基本问题的发生意味着程序不能继续运行了。ok() 方法返回了一个值，在这个值上定义了另外一个方法： expect()。 expect() 方法调用时，会获取一个值，如果这个值不是正确的值，那么会导致一个应急（panic！）的发生，你要传递一个信息给这个应急。这样的一个应急(panic！)会导致我们的程序崩溃，并且会显示你传递的信息。

如果我们不调用这两个方法，程序能编译成功，但我们会得到一个警告信息：

	$ cargo build
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	src/main.rs:10:5: 10:39 warning: unused result which must be used,
	#[warn(unused_must_use)] on by default
	src/main.rs:10     io::stdin().read_line(&mut guess);
	                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Rust 警告我们，我们没有使用 Result 值。这个警告来自 io：：Result 所有的一个特殊的注释。Rust 试图告诉你，你没有处理一个有可能的错误。避免这个错误的正确方法是写好错误的处理部分的代码。幸运的是，如果程序有问题，而我们只是让程序自己崩溃就好，那么我们可以用这两个方法。但是如果我们想让程序能从错误中恢复，那么我们就要做点其他的事情了。这部分内容我们保留到未来的一个项目里再讲。

第一个例子就剩下这一行代码了：

	println！("You guessed： {}"， guess);
	}

这行代码打印出我们输入的字符串。{} 是一个占位符，所以我们把 guess  作为参数传递给它。如果我们有多个 {} ，那么我们将传递多个参数：

	let x = 5;
	let y = 10;
	
	println!("x and y: {} and {}", x, y);

容易吧。

不管怎样，这就是我们的学习过程啦。我们可以用我们所拥有的 cargo run 来运行代码了：

	$ cargo run
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	     Running `target/debug/guessing_game`
	Guess the number!
	Please input your guess.
	6
	You guessed: 6

好吧！我们的第一部分已经完成了：我们可以从键盘输入，然后将它打印出来。

##生成一个秘密数字

接下来，我们需要生成一个秘密数字。Rust 还未引入有随机数方法的标准库。然而，Rust 的团队提供了一个随机箱 [rand crate](https://crates.io/crates/rand)。一个“箱”是一个 Rust 的代码包。我们已经构建了一个可执行的“二进制箱”。 rand 是一个“箱库”，其中包含的代码可以在其他程序中使用。

使用外部箱是 Cargo 的一个闪光点。在我们使用 rand 写代码之前，我们需要修改 Cargo.toml。打开它，并在文件底部添加这几行：

	[dependencies]
	
	rand="0.3.0"

Cargo.toml 的 [dependencies] 的部分就像 [package] 部分一样：直到下一个部分开始之前，它下面的内容都是它所包含的。Cargo 通过使用依赖关系部分来知道你所拥有的依赖外部箱，以及你需要的版本。在本例中，我们使用的版本是 0.3.0。Cargo 能理解语义版本([Semantic Versioning](http://semver.org/))，这是一个编写版本号的标准。如果我们想使用最新的版本，我们可以使用 * ，或者我们也可以使用一个范围的版本。Cargo 的[文档](http://doc.crates.io/crates-io.html)包含更多的细节内容。

现在，在不改变我们的任何代码，让我们重新构建我们的项目：

	$ cargo build
	    Updating registry `https://github.com/rust-lang/crates.io-index`
	 Downloading rand v0.3.8
	 Downloading libc v0.1.6
	   Compiling libc v0.1.6
	   Compiling rand v0.3.8
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)

(你可以看到不同版本)。

有大量的新的内容出现了！现在我们有一个外部依赖，Cargo 从注册表获取所有依赖的最新版本，我们这里的注册表是一个来自 Crates.io 的数据副本。这里的 Crates.io 是 Rust 生态系统中人们发布给他人使用的开源 Rust 项目的地方。

更新注册表，Cargo 会检查我们的 [dependencies] 并下载我们没有的一些依赖。在本例中，尽管我们说我们只想依赖 rand，但是我们也获取了一份 libc 的拷贝。这是因为 rand 是基于 libc 工作的。下载完成后，Cargo 会先编译这些依赖项，然后再编译我们的项目。

如果我们再次运行 cargo build，我们将会得到不同的输出：

	$ cargo build

没有错，这里是没有输出的！Cargo 知道我们的项目已经构建完成，并且项目所有的依赖项也构建完成，因此没有理由将所有这些事情再做一遍。既然无事可做，它就退出了。如果我们再次打开 src/main.rs，做一个微不足道的改变，然后再保存它，再次运行，我们就会看到一行：

	$ cargo build
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)

所以，我们告诉 Cargo 我们想要任何 0.3.x 版本的 rand，因此它会获取当时的最新版本，v0.3.8。但是当下周有重要错误修正的版本 v0.3.9 出现时会发生什么？虽然错误修正是很重要的，但是如果 0.3.9 版本包含会破坏我们代码的一个回归呢？

这个问题的答案是锁文件 Cargo.lock，现在你可以在你的项目目录中找到它。当你第一次构建你的项目时，Cargo 会找出所有的符合你的要求的版本，然后将他们写到 Cargo.lock 文件中。当未来你构建项目时，Cargo将会发现 Cargo.lock 文件的存在，然后使用文件指定的版本，而不是再次做找出版本的工作。这可以让你有一个可重复的自动构建。换句话说，直到我们明确升级之前，我们将待在 0.3.8 版本。所以那些分享我们的代码人，也会感谢锁文件的存在。

那么当我们真的想使用 v0.3.9 版本时，怎么办？Cargo 还有另外一个命令， update，它的意思是“忽略锁文件，找出适合我们指定要求的所有最新版本。如果可行，那么将那些版本信息写到锁文件中。”但是，默认情况下，Cargo 只会比寻找大于 0.3.0 并且小于 0.4.0 的版本。如果我们想升级到 0.4.x 版本，那我们必须直接更新 Cargo.toml 文件。如果我们这么做了，我们下次运行 cargo build 时，Cargo 将更新索引并重新评估我们对 rand 的需求。

这里提到了很多关于 [Cargo](http://doc.crates.io/) 和它的[生态系统](http://doc.crates.io/crates-io.html)的内容，对现在来说，这就是我们所需要知道关于他们的全部信息。Cargo 使 Rust 很容易重复使用库，所以 Rustaceans 倾向于编写当做子包组装的小项目。

让我们实际使用一下 rand。下面是我们要进行的下一个步骤：


	extern crate rand;
	
	use std::io;
	use rand::Rng;
	
	fn main() {
	    println!("Guess the number!");
	
	    let secret_number = rand::thread_rng().gen_range(1, 101);
	
	    println!("The secret number is: {}", secret_number);
	
	    println!("Please input your guess.");
	
	    let mut guess = String::new();
	
	    io::stdin().read_line(&mut guess)
	        .ok()
	        .expect("failed to read line");
	
	    println!("You guessed: {}", guess);
	}


我们所做的第一件事就是改变了第一行代码。现在第一行是，extern crate rand。因为我们在我们的 [dependencies] 中声明了 rand，所以我们可以使用 extern crate 让 Rust 知道我们会使用它。这也相当于一个 use rand;，所以我们可以通过加前缀 rand：：来利用 rand 箱中的所有东西。

接下来，我们添加了另一个 use 行： use rand：：Rng。一会儿我们要用一种方法，它需要 Rng 在作用域内才运行。基本的思路是这样的：方法是定义在一些所谓的“特征”上的，它需要特征在域内，才能使方法工作。更多的细节内容，请阅读特征章节。

在中间，有我们添加的另外两行：

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

我们使用 rand：：thread_rng() 函数获取随机数产生器的一个副本，这是我们在执行的一个本地线程。我们上面使用 use rand：：Rng，它有一个可用的 gen\_range() 方法。这个方法有两个参数，生成一个在两个参数范围内的数字。它包括下界值，但不包括上界值，所以我们需要 1 和 101 两个参数，从而来获取一个一到一百之间的数字。

第二行打印出了这个生成的秘密数字。这是在我们开发项目时是非常有用的，我们可以很容易地对其进行测试。但是在最终版本里我们会删除它。如果一开始的时候就打印出来你要猜的数字，显然这不是一个游戏！

尝试运行几次我们的程序：
	
	$ cargo run
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	     Running `target/debug/guessing_game`
	Guess the number!
	The secret number is: 7
	Please input your guess.
	4
	You guessed: 4
	$ cargo run
	     Running `target/debug/guessing_game`
	Guess the number!
	The secret number is: 83
	Please input your guess.
	5
	You guessed: 5

太棒了！下一篇：让我们用我们的猜测值和秘密数字比较一下。

##比较猜测值

现在，我们已经得到了用户的输入，让我们将我们猜测值与随机值比较下。下面是我们要进行的下一步，它现在还不能运行：

	extern crate rand;
	
	use std::io;
	use std::cmp::Ordering;
	use rand::Rng;
	
	fn main() {
	    println!("Guess the number!");
	
	    let secret_number = rand::thread_rng().gen_range(1, 101);
	
	    println!("The secret number is: {}", secret_number);
	
	    println!("Please input your guess.");
	
	    let mut guess = String::new();
	
	    io::stdin().read_line(&mut guess)
	        .ok()
	        .expect("failed to read line");
	
	    println!("You guessed: {}", guess);
	
	    match guess.cmp(&secret_number) {
	        Ordering::Less    => println!("Too small!"),
	        Ordering::Greater => println!("Too big!"),
	        Ordering::Equal   => println!("You win!"),
	    }
	}


这里的有几个新部分。第一个是另外的一个 use。我们将叫做 std：：cmp：：Ordering 的类型放到了作用域内。然后是底部新的五行代码：

	match guess.cmp(&secret_number) {
	    Ordering::Less    => println!("Too small!"),
	    Ordering::Greater => println!("Too big!"),
	    Ordering::Equal   => println!("You win!"),
	}

 cmp() 方法可以在任何可比较的地方被调用，但是它需要你想要比较东西的一个引用。它将返回我们之前 use 的 Ordering 类型。我们使用一个 match 语句来确定它到底是一个什么样的 Ordering。 Ordering 是一个“枚举”（enum，enumeration的简称）。枚举看起来像这个样子的：

	enum Foo {
	    Bar，
	    Baz，
	}

利用这个定义，任何类型是 Foo 的类型，可以是一个 Foo：：Bar 或者是一个 Foo：：Baz。我们使用 ：：表明一个特定 enum 变量的命名空间。

Ordering 枚举有三个可能的变量： Less， Equal 和 Greater。  match 语句需要有一个类型的值输入，并允许你为每一可能的值创建一个“手臂”。因为我们有三种类型的 Ordering，因此我们要三个：

	match guess.cmp(&secret_number) {
	    Ordering::Less    => println!("Too small!"),
	    Ordering::Greater => println!("Too big!"),
	    Ordering::Equal   => println!("You win!"),
	}

如果它是 Less，我们打印 Too small！，如果是 Greater，打印 Too big！，如果 Equal，打印 You win！。 match 是非常有用的，在 Rust 中经常会使用它。

我之前提到了，项目现在还不能运行。让我们来试一试：

	$ cargo build
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	src/main.rs:28:21: 28:35 error: mismatched types:
	 expected `&collections::string::String`,
	    found `&_`
	(expected struct `collections::string::String`,
	    found integral variable) [E0308]
	src/main.rs:28     match guess.cmp(&secret_number) {
	                                   ^~~~~~~~~~~~~~
	error: aborting due to previous error
	Could not compile `guessing_game`.


唷！这是一个很严重的错误。它核心的意思是，我们有不匹配的类型。Rust 有一个强壮的、静态的类型系统。然而，它也有类型推导过程。当我们写了  let guess = String：：new()，Rust 能够推断出 guess 应该是一个 String，所以它不用让我们写出 guess 的类型。然而值从 1 到 100 的 secret\_number 的类型就有很多了：i32，32 位的数字，u32，32 位无符号数字，或者 i64，一个 64 位的数字，或其的一些类型。Rust 的默认类型是 i32。到目前为止，还没有什么关系。然而在比较代码中，Rust 不知道如何比较 guess 和 secret\_number。他们需要是相同的类型。最终，我们把读入的 String 转换成一个数字类型，来进行比较。我们用额外的三行来做这件事情。下面是我们的新程序：

	extern crate rand;
	
	use std::io;
	use std::cmp::Ordering;
	use rand::Rng;
	
	fn main() {
	    println!("Guess the number!");
	
	    let secret_number = rand::thread_rng().gen_range(1, 101);
	
	    println!("The secret number is: {}", secret_number);
	
	    println!("Please input your guess.");
	
	    let mut guess = String::new();
	
	    io::stdin().read_line(&mut guess)
	        .ok()
	        .expect("failed to read line");
	
	    let guess: u32 = guess.trim().parse()
	        .ok()
	        .expect("Please type a number!");
	
	    println!("You guessed: {}", guess);
	
	    match guess.cmp(&secret_number) {
	        Ordering::Less    => println!("Too small!"),
	        Ordering::Greater => println!("Too big!"),
	        Ordering::Equal   => println!("You win!"),
	    }
	}

新的三行代码：

    let guess: u32 = guess.trim().parse()
        .ok()
        .expect("Please type a number!");

等一下，我想我们已经有了一个 guess 了吧?对，我们确实有一个，但是 Rust 允许我们用一个新的 guess 来“遮住”前面的那一个。在具体情况中这经常使用。 guess 开始时是一个 String，但我们想把它转换成一个  u32。遮蔽功能能让我们重复使用 guess 这个名字，而不是迫使我们使用两个唯一的名字如 guess\_str ，guess，或者其他之类的。

类似于我们之前写的代码那样，我们将 guess 绑定到一个表达式：

	guess.trim().parse()

紧随其后的是一个 ok().expect() 的调用。这里的 guess 指的是旧的那个 guess，也就是我们输入 String 到其中那一个。String 的 trim() 方法是消除 String 前端和后端所有空白字符串的方法。这是很重要的，因为我们必须按“回车”键来结束 read_line() 方法的输入。这意味着，如果我们键入 5 并且敲击回车键，guess 看起来是这样的：5\n。\n 表示“换行符”，回车键。trim() 方法就是用来去除类似这样的东西的，让我们的字符串只有 5。 字符串的 [parse() ](http://doc.rust-lang.org/stable/std/primitive.str.html#method.parse)方法将字符串解析成某类数字。因为它可以解析各种数据，所以我们需要给 Rust 提示我们想要转化的确切数字类型。因此这样写，let guess： u32。guess 后面的冒号(：) 告诉 Rust 我们要转化的类型。 u32 是一个无符号 32 位整数。Rust 有许多[内置数字类型](http://doc.rust-lang.org/stable/book/primitive-types.html#numeric-types)，这里我们选择了 u32。对于一个小的正整数，这是一个很好的默认选择。

就像 read\_line()，我们的 parse() 调用可能会导致一个错误。如果我们的字符串包含了 A% 呢？这就没有办法转换成一个数字了。因此，我们会像 read\_line() 做的那样：用 ok() 和 expect() 方法去处理崩溃，如果有一个错误发生了的话。

让我们来试试我们的新程序！

	$ cargo run
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	     Running `target/guessing_game`
	Guess the number!
	The secret number is: 58
	Please input your guess.
	  76
	You guessed: 76
	Too big!

好了！你可以看到我甚至在猜的数字之前添加了空格，程序仍然发现我猜的数是 76。运行几次程序，再猜一个比较小的数字，验证猜测程序能正常运行。

现在我们已经做了游戏的大部分工作，但我们只能做一个猜测。让我们通过添加循环来改变这一情况！

##循环

loop 关键字为我们提供了一个无限循环。让我们添加如下的代码并尝试一下：

	extern crate rand;
	
	use std::io;
	use std::cmp::Ordering;
	use rand::Rng;
	
	fn main() {
	    println!("Guess the number!");
	
	    let secret_number = rand::thread_rng().gen_range(1, 101);
	
	    println!("The secret number is: {}", secret_number);
	
	    loop {
	        println!("Please input your guess.");
	
	        let mut guess = String::new();
	
	        io::stdin().read_line(&mut guess)
	            .ok()
	            .expect("failed to read line");
	
	        let guess: u32 = guess.trim().parse()
	            .ok()
	            .expect("Please type a number!");
	
	        println!("You guessed: {}", guess);
	
	        match guess.cmp(&secret_number) {
	            Ordering::Less    => println!("Too small!"),
	            Ordering::Greater => println!("Too big!"),
	            Ordering::Equal   => println!("You win!"),
	        }
	    }
	}

先等一下，我们只是添加一个无限循环么？是的，没错。还记得我们关于 parse() 的讲解吗？如果我们给出了一个非数字的回答，它将会 return 并且退出。我们来看：

	$ cargo run
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	     Running `target/guessing_game`
	Guess the number!
	The secret number is: 59
	Please input your guess.
	45
	You guessed: 45
	Too small!
	Please input your guess.
	60
	You guessed: 60
	Too big!
	Please input your guess.
	59
	You guessed: 59
	You win!
	Please input your guess.
	quit
	thread '<main>' panicked at 'Please type a number!'

哈！像其他非数字的输入一样，quit 确实是退出了。嗯，至少可以说这是一个次优的解决方案。第一个方案：当你赢得比赛时，程序退出：

	extern crate rand;
	
	use std::io;
	use std::cmp::Ordering;
	use rand::Rng;
	
	fn main() {
	    println!("Guess the number!");
	
	    let secret_number = rand::thread_rng().gen_range(1, 101);
	
	    println!("The secret number is: {}", secret_number);
	
	    loop {
	        println!("Please input your guess.");
	
	        let mut guess = String::new();
	
	        io::stdin().read_line(&mut guess)
	            .ok()
	            .expect("failed to read line");
	
	        let guess: u32 = guess.trim().parse()
	            .ok()
	            .expect("Please type a number!");
	
	        println!("You guessed: {}", guess);
	
	        match guess.cmp(&secret_number) {
	            Ordering::Less    => println!("Too small!"),
	            Ordering::Greater => println!("Too big!"),
	            Ordering::Equal   => {
	                println!("You win!");
	                break;
	            }
	        }
	    }
	}

通过在打印 You win！的代码后面添加一行 break，当我们赢了的时候就会退出循环。退出循环也意味着退出了程序，因为它是 main() 做的最后一件事。这里我们想做一个额外的调整：当有人输入非数字时，我们并不退出，只是忽略它。我们可以这样做：

	extern crate rand;
	
	use std::io;
	use std::cmp::Ordering;
	use rand::Rng;
	
	fn main() {
	    println!("Guess the number!");
	
	    let secret_number = rand::thread_rng().gen_range(1, 101);
	
	    println!("The secret number is: {}", secret_number);
	
	    loop {
	        println!("Please input your guess.");
	
	        let mut guess = String::new();
	
	        io::stdin().read_line(&mut guess)
	            .ok()
	            .expect("failed to read line");
	
	        let guess: u32 = match guess.trim().parse() {
	            Ok(num) => num,
	            Err(_) => continue,
	        };
	
	        println!("You guessed: {}", guess);
	
	        match guess.cmp(&secret_number) {
	            Ordering::Less    => println!("Too small!"),
	            Ordering::Greater => println!("Too big!"),
	            Ordering::Equal   => {
	                println!("You win!");
	                break;
	            }
	        }
	    }
	}

这些代码改变了：

	let guess: u32 = match guess.trim().parse() {
	    Ok(num) => num,
	    Err(_) => continue,
	};

通过将使用 ok().expect() 换做使用一个 match 语句，这可以教会你是如何从“崩溃错误”转变到“处理错误”的。parse() 返回的 Result 是一个像 Ordering 一样的枚举，但在这里，每个变量都有与之相关的一些数据： Ok 是一个 success， Err 是一个 failure。它们每个又包含更多的信息：成功的解析成整数，一个错误的类型。在这种情况下，我们将匹配 match 在 Ok(num)，这里将 Ok 的内部值的名字设置为 num，然后我们在右侧返回它的值。在 Err 的情况下，我们不关心它是什么样的错误，所以这里我们只用一个下划线 ‘_’，而不是一个名字。这样会忽略掉错误，continue 会继续进行下一个循环(loop)的迭代。

现在，问题应该都解决了！让我们试一试：
	
	$ cargo run
	   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
	     Running `target/guessing_game`
	Guess the number!
	The secret number is: 61
	Please input your guess.
	10
	You guessed: 10
	Too small!
	Please input your guess.
	99
	You guessed: 99
	Too big!
	Please input your guess.
	foo
	Please input your guess.
	61
	You guessed: 61
	You win!

太棒了！经过最后一个小小的调整，我们已经完成了猜谜游戏。你能想到什么？没错，我们不想打印出秘密数字。打印这个数字对于测试来说是很棒的一件事，但是当运行时，它会毁掉这个游戏的。下面是我们最终的代码：

	extern crate rand;
	
	use std::io;
	use std::cmp::Ordering;
	use rand::Rng;
	
	fn main() {
	    println!("Guess the number!");
	
	    let secret_number = rand::thread_rng().gen_range(1, 101);
	
	    loop {
	        println!("Please input your guess.");
	
	        let mut guess = String::new();
	
	        io::stdin().read_line(&mut guess)
	            .ok()
	            .expect("failed to read line");
	
	        let guess: u32 = match guess.trim().parse() {
	            Ok(num) => num,
	            Err(_) => continue,
	        };
	
	        println!("You guessed: {}", guess);
	
	        match guess.cmp(&secret_number) {
	            Ordering::Less    => println!("Too small!"),
	            Ordering::Greater => println!("Too big!"),
	            Ordering::Equal   => {
	                println!("You win!");
	                break;
	            }
	        }
	    }
	}

##完成了！

此时此刻，你已经成功地构建了猜谜游戏！恭喜你！

这第一个项目向你展示了很多内容： let， match，方法，相关函数，使用外部箱等等。我们的下一个项目将展示更多的内容。