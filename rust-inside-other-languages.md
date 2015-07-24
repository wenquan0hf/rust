# Rust 嵌入到其他语言

我们的第三个项目，我们要选择展示那些能展示 Rust 最大优点的点：大量运行时的减少。

随着我们组织的发展，其越来越依赖其他的一些编程语言。不同的编程语言有不同的优点和缺点，通晓数种语言的堆栈允许你使用一个特定的语言，在其的优势方面,而在其弱势的方面，你可以使用另一种语言。

许多程语言一个共同薄弱的地方就是程序的运行时性能。通常情况下，使用了一种运行比较慢的语言，但是如果它同时能提升程序员的工作效率也是值得的。为了帮助缓解这个问题,他们提供了一个方法，系统中的一部分用 C 来写，然后再调用 C 代码，那么这一部分好像就是用高级语言编写的似得。这被称作“外部程序接口”，一般缩写成“FFI”。

Rust 在两个方面上支持 FFI:它可以容易的调用 C 代码，但至关重要的是，它也可以像容易调用 C 代码那样被调用。当你需要一些额外的一些其他功能时，Rust 的无垃圾收集器和较的低运行时需求，这两点使得 Rust 成为一个嵌入到其他语言中的很好的方案。

在本教程中，我们有一整章来讲述 FFI 和它的细节，但是在本章中，我们将用三个例子来展示 FFI 的特定用例，它们分别是在 Ruby，Python 和 JavaScript 中。

## 问题

这里我们有很多不同的项目可供选择，但我们要选择一个能展示 Rust 比其他许多语言有明显优势的例子：数值计算和线程。

许多语言为了一致性，将数字存放在堆上，而不是在堆栈上。尤其是在专注于面向对象编程和使用垃圾收集的语言上，默认的分配模式是堆分配。有时候优化会将特定的数字分配给堆栈，但它不是依靠优化器来完成的这项工作。同时，我们可能希望确保我们使用的总是原始的数字类型而不是某种形式的对象类型。

第二问题，许多语言有一个“全局解释器锁”，这在许多情况下限制了并发。这是以安全的名义来进行的，本来这是一个好意，但是它限制了同一时间可以完成的工作量，这就非常不好了。

为了强调这两个方面，我们要创建一个能使用到这两方面的一个小项目。因为示例的重点是将 Rust 嵌入到其他语言，而不是这个问题的本身，所以我们只用一个玩具例子:

```
	开十个线程。每个线程内部实现从一数到五百万。数完后，十个线程结束，并打印出“done!”。
```

这里我基于我计算机的能力选择了五百万。下面是一个用 Ruby 写的例子的代码:

```
	threads = []
	
	10.times do
	  threads << Thread.new do
	    count = 0
	
	    5_000_000.times do
	      count += 1
	    end
	  end
	end
	
	threads.each {|t| t.join }
	puts "done!"
```

尝试运行这个例子，并选择一个数字运行几秒钟。基于你的电脑硬件，你可能要增大或减小这个数字。

在我的系统中，运行这个程序需要 2.156 秒。如果我用某种类似 top 的进程监控工具，我可以看到它在我的机器上只使用一个 CPU 核。这是由于 GIL 在起作用。

虽然这确实是一个人工合成的程序，但是你也可以想象许多与现实世界相似的问题。就我们的目的而言，运行一些繁忙线程就代表了某种并行，昂贵的计算问题。

## Rust 库

让我们用 Rust 重写这个问题。首先，让我们用 Cargo 创建一个新项目:
	
```
	$ cargo new embed
	$ cd embed
```

这个程序用 Rust 写起来很简单:

```
	use std::thread;
	
	fn process() {
	    let handles: Vec<_> = (0..10).map(|_| {
	        thread::spawn(|| {
	            let mut _x = 0;
	            for _ in (0..5_000_001) {
	                _x += 1
	            }
	        })
	    }).collect();
	
	    for h in handles {
	        h.join().ok().expect("Could not join a thread!");
	    }
	}
```

这个程序中的一些内容与从先前的例子看起来很相似。我们开启了十个线程，将它们收集成一个 handles 向量。在每个线程中，我们都循环五百万次，每次给 `_x` 加一。这里为什么使用下划线？嗯，如果我们删除了它并编译:

```
	$ cargo build
	   Compiling embed v0.1.0 (file:///home/steve/src/embed)
	src/lib.rs:3:1: 16:2 warning: function is never used: `process`, #[warn(dead_code)] on by default
	src/lib.rs:3 fn process() {
	src/lib.rs:4     let handles: Vec<_> = (0..10).map(|_| {
	src/lib.rs:5         thread::spawn(|| {
	src/lib.rs:6             let mut x = 0;
	src/lib.rs:7             for _ in (0..5_000_001) {
	src/lib.rs:8                 x += 1
	             ...
	src/lib.rs:6:17: 6:22 warning: variable `x` is assigned to, but never used, #[warn(unused_variables)] on by default
	src/lib.rs:6             let mut x = 0;
	                             ^~~~~
```

第一个警告是因为我们正在构建一个库。如果我们有一个关于此函数的测试，那么这个警告将会消失。但是现在，这个函数从未被调用过。

第二个警告与 x ，`_x` 有关。因为我们对 x 没有做过任何操作，所以我们得到了一个警告。在我们的例子中，这是完全没有问题的，因为我们就是想浪费 CPU 周期。为 x 添加下划线前缀就会消除这个警告。

最后，我们连接了每个线程。

然而现在这是一个 Rust 库，它还没有公开任何从 C 中可调用的代码。如果现在我们试图将其链接到另一种语言，那么它是不能使用的。我们只需要做两个小改变就能解决这个问题。第一个就是修改我们代码的开始部分:

```
	#[no_mangle]
	pub extern fn process() {
```

我们必须添加一个新的属性，no_mangle。当你创建一个 Rust 库时，在编译输出阶段会改变函数的名称。这个的原因超出了本教程的范围。为了让其他语言知道如何调用函数，我们不需要改变函数的名称。这个属性就是将这个改变功能关掉。

另一个变化就是 pub extern。pub 意味着在这个模块以外这个函数应该是可调用的。extern 表示它应该能够从 C 中被调用。就这样了!没有很多变化了。

我们需要做的第二件事就是是改变 Cargo.toml 的设置。添加如下的内容到底部:

```
	[lib]
	name = "embed"
	crate-type = ["dylib"]
```

这会告诉 Rust，我们想将我们的库编译成标准动态库。默认情况下，Rust  会编译成一个 ‘rlib’，这是一个 Rust 独有的格式。

现在让我们来构建项目:

```
	$ cargo build --release
	   Compiling embed v0.1.0 (file:///home/steve/src/embed)
```

我们选择最优化的构建方式 `cargo build --release`。因为我们希望程序能运行的尽可能快！你可以从 `target/release` 中找到库文件的输出:

```
	$ ls target/release/
	build  deps  examples  libembed.so  native
```

这里的 libembed.so 是我们的‘共享对象’库。我们可以像用 C 写的对象库一样使用这个文件!说句题外话，根据平台的不同，这个文件可能是 embed.dll 或是 libembed.dylib。

现在我们已经构建了我们的 Rust 库，让我们在 Ruby 使用它吧。

## Ruby

在我们的项目中打开 embed.rb 文件，并且按如下所做：

```
	require 'ffi'
	
	module Hello
	  extend FFI::Library
	  ffi_lib 'target/release/libembed.so'
	  attach_function :process, [], :void
	end
	
	Hello.process
	
	puts "done!”
```

在我们可以运行这个程序之前，我们要先安装 ffi gem：

```
	$ gem install ffi # this may need sudo
	Fetching: ffi-1.9.8.gem (100%)
	Building native extensions.  This could take a while...
	Successfully installed ffi-1.9.8
	Parsing documentation for ffi-1.9.8
	Installing ri documentation for ffi-1.9.8
	Done installing documentation for ffi after 0 seconds
	1 gem installed
```

最终，我们可以试着运行一下：

```
	$ ruby embed.rb
	done!
	$
```

哇，好快！在我的系统上，这花费了 0.086 秒的时间，而不是纯 Ruby 版本花费的两秒钟。让我们详细讲一下这段 Ruby 代码：

```
	require 'ffi
```

首先我们需要 ffi gem。这能让我们像连接 C 库一样与 Rust 库连接。

```
	module Hello
	  extend FFI::Library
	  ffi_lib 'target/release/libembed.so'
```

ffi gem 的作者推荐使用一个模块来圈定我们将要从共享库中导入的方法的作用域。在模块里面,我们 extend 了必要的 FFI::Library 库模块，然后调用 `ffi_lib` 来加载共享对象库。我们只是给它传递了我们库文件的存储路径，正如我们之前看到的，这个路径是 `target/release/libembed.so`。

```
	attach_function :process, [], :void
```

`attach_function` 方法是由 FFI gem 提供的。它是用来连接 Rust 与 Ruby 中同名函数 process() 的。因为 process() 不需要任何参数，所以第二个参数是一个空数组。因为它不返回任何东西，所以我们传递 `:void `作为最后的一个参数。

```
	Hello.process
```

这才是对 Rust 的实际调用。我们的 module 和 `attach_function` 的调用结合才成就了它。它看起来像是一个 Ruby 函数，但实际上是 Rust 的!

	puts "done!"

最后，根据我们之前的项目的需求，我们打印出 done!

就是它了!正如我们所看到的，两种语言之间的桥接是真的很容易，并且提升了很多性能。

接下来，让我们来试试 Python 吧!

## Python

在目录中创建一个 embed.py 文件，并将如下内容输入：

```
	from ctypes import cdll
	
	lib = cdll.LoadLibrary("target/release/libembed.so")
	
	lib.process()
	
	print("done!")
```

这个更简单！我们用 ctypes 模块中的 cdll。稍后对 LoadLibrary 的一个快速调用后，我们就可以调用 process()了。

在我的系统中，这花费了 0.017 秒。快吧！

## Node.js

Node 不是一门语言，但它目前是服务器端 JavaScript 的主要实现。

为了用 Node 实现 FFI，我们首先需要安装库:

```
	$ npm install ffi
```

安装完成后，我们就可以使用它了：

```
	var ffi = require('ffi');
	
	var lib = ffi.Library('target/release/libembed', {
	  'process': [ 'void', []  ]
	});
	
	lib.process();
	
	console.log("done!");
```

与 Python 的例子相比，它看起来更像 Ruby 的例子。我们用 ffi 模块获得 ffi.Library()，它负责加载共享对象。我们要解释下函数的返回类型和参数类型，返回的是‘空’，参数是一个空数组来表示的。从此之后，我们就可以调用它并打印结果。

在我的系统中，程序运行花费了 0.092 秒。

## 结论

正如你可以看到的，这些基本操作是非常简单的。当然，这里我们还有很多可以做的。更多的细节请查看 FFI 章节。
