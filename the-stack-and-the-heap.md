## 栈和堆

作为一种系统语言，Rust 运行在较低的层次。如果你只学习过高级语言，有一些系统编程方面的问题，你可能不熟悉。最重要的一个问题是存储器如何工作，例如如何使用堆和栈。如果你对 c 语言如何使用堆栈分配熟悉的话，本章将会是一个复习。如果你不熟悉的话，你将会学习到Rust-y 关注的一些相关基本概念。

### 内存管理

关于内存管理有两个常用术语。栈和堆是一种抽象概念，帮助您确定何时分配和释放内存。

这里有一个更高层次的比较：

栈是非常快的，也是 Rust 默认的内存分配方式。但是分配存在于本地函数调用，且在大小方面是有限的。另一方面，堆的速度相对比较慢，但是你的程序可以明确地分配堆内存。且它实际上是无限制的，可以在全局范围内访问。

### 栈

让我们谈谈这个 Rust 程序:

    fn main() {
    	let x = 42;
    }

这个程序有一个变量绑定 x。这需要分配内存。默认情况下，Rust 进行栈分配，这意味着基值存储到栈内。这是什么意思呢?

当一个函数被调用时，一些内存被分配给它的所有局部变量和其他一些信息。这就是所谓的“栈帧”，本教程的目的，我们将忽略额外的信息，只考虑局部变量的分配。所以在这种情况下，当 main() 运行时，将为我们的栈帧分配一个 32 位整数。如你所见，这是自动处理的，我们不必为此编写任何特殊 Rust 代码或其他任何东西。

函数结束后，它的栈帧被收回。这也是自动实现的，我们不需要做什么特别的事情。

这就是一个简单的程序全部的内容。关键的是要理解栈分配是非常，非常快的。在我们了解所有的局部变量之前，我们有时间可以一次性抓取内存。并且因为我们可以在同一时间丢弃他们，我们也可以迅速摆脱它。

不利的一面是，当我们不只是在一个单一的函数内需要它们时，却不能长存这些值。我们还没有谈到这个名字：“栈”的意思。为此,我们需要一个稍微复杂一点的例子：

    fn foo() {
    	let y = 5;
    	let z = 100;
    }
    
    fn main() {
    	let x = 42;
    
    	foo();
    }

这个程序总共有三个变量：两个在 **foo()** 中,一个在 **main()** 中。和之前一样，当 **main()** 被调用时，单个整数分配它的栈帧。但是在此之前可以显示调用 **foo()** 时会发生什么，我们需要想象内存中是如何操作的。操作系统提供了一个程序的内存视图，它非常简单：一个巨大的地址列表，从 0 到很大的值，代表你的计算机有多少内存。例如，如果你有一个 G 的内存，你的地址从 **0** 到 **1,073,741,824**。这一数字来自于 2 的 30 次方，是一个十亿字节的数字级别。

这个内存就像是一个巨大的数组：地址从 0 开始，到最后的数。这是第一个栈帧的图表：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

我们可以知道地址为 **0** 的地方存储了一个名称为 **x**，值为 **42** 的东西。

调用 **foo()**，一个新的栈帧被分配：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2</td>
<td>z</td>
<td>100</td>
</tr>
<tr>
<td>1</td>
<td>y</td>
<td>5</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

因为 **0** 被第一帧占用，**1** 和 **2** 用于 **foo()** 的栈帧。我们调用的函数越多，内存向上增长的越多。

这里，我们必须注意一些重要的事情。数字 0、1 和 2 都仅仅是为了便于说明，和计算机实际使用的情况没有丝毫关系。特别是，是现实中，一系列地址是会被一些单独的自己隔开成单独的每个地址的，这些分割甚至可能会超过存储的值的大小。

**foo()** 结束后，其帧被收回：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

然后，**main()** 结束后，甚至最后的值都会消除。简单吧!

这就是所谓的“栈”，因为它像一个餐盘的栈：你放下去的第一块盘子将是你最后拿回来的盘子。栈有时被称为“后进先出队列”的原因，就是你最后放入栈内的值就是第一个你需要检索的值。

让我们尝试一个三层的例子：

    fn bar() {
    	let i = 6;
    }
    
    fn foo() {
    	let a = 5;
    	let b = 100;
    	let c = 1;
    
    	bar();
    }
    
    fn main() {
    	let x = 42;
    
    	foo();
    }

好吧，首先，我们调用 **main()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

接下来，**main()** 调用 **foo()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>3</td>
<td>c</td>
<td>1</td>
</tr>
<tr>
<td>2</td>
<td>b</td>
<td>100</td>
</tr>
<tr>
<td>1</td>
<td>a</td>
<td>5</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

然后 **foo()** 调用 **bar()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>4</td>
<td>i</td>
<td>6</td>
</tr>
<tr>
<td>3</td>
<td>c</td>
<td>1</td>
</tr>
<tr>
<td>2</td>
<td>b</td>
<td>100</td>
</tr>
<tr>
<td>1</td>
<td>a</td>
<td>5</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

现在，我们的栈越来越高了。

**bar()** 结束后，其栈帧被回收，留下 **foo()** 和 **main()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>3</td>
<td>c</td>
<td>1</td>
</tr>
<tr>
<td>2</td>
<td>b</td>
<td>100</td>
</tr>
<tr>
<td>1</td>
<td>a</td>
<td>5</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

然后 **foo()** 结束后，只留下 **main()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>42</td>
</tr>
</table>

然后我们就大功告成了。明白了吗？就像堆餐具：一直添加到顶部，然后从顶部开始取走。

### 堆

这种方法可以很好地工作，但并不是一切事物都可以像这样工作。有时，不同功能之间需要共享内存，或者不只是在单个函数的执行期间保持活动状态。为此，我们可以使用堆。

在 Rust 中，你可以使用 [Box<T> 类型](https://doc.rust-lang.org/stable/std/boxed/)在堆上分配内存。这里有一个例子：

    fn main() {
    	let x = Box::new(5);
    	let y = 42;
    }

当 **main()** 被调用时，内存发生如下变化：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>1</td>
<td>y</td>
<td>42</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>??????</td>
</tr>
</table>

我们在栈上为两个变量分配空间。像以往一样，**y** 值为 **42**，但 **x** 呢？恩，**x** 是一个 **Box<i32>** 类型的值，**Boxes** 在堆上分配内存。**boxes** 的实际值是一种数据结构，实际上是指向“堆”的一个指针。当我们开始执行函数，调用 **Box::new()**，它为堆分配一些内存，将 **5** 放在内存内。内存现在看起来像这样：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30的方</td>
<td></td>
<td>5</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>1</td>
<td>y</td>
<td>42</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>2的30的方</td>
</tr>
</table>

在一个假想的电脑里，我们有 1GB 的 RAM。因为我们的栈从零增长，从最简单的地方开始分配内存。所以我们的第一个值是在内存中最高的地方。x 这个结构的值有一个[原始指针](https://doc.rust-lang.org/stable/book/raw-pointers.html)，指向在堆上分配的位置，所以 x 的值是 2 的 30 次方，即我们请求的内存位置。

关于在这些环境中实际上如何进行分配和释放内存，我们还没有谈论太多。本教程中，我们会讨论更深的细节，但这里必须指出的是，堆不像栈是从一端开始生长。在本书后边的部分，我们会讨论到这样的一个例子，但是由于堆可以以任何顺序分配和释放内存，最终以很多“洞”的存在结束。这是一个现在已经运行了一段时间的程序的内存布局图:

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30的方</td>
<td></td>
<td>5</td>
</tr>
<tr>
<td>2的30的方-1</td>
<td></td>
<td></td>
</tr>
<tr>
<td>2的30的方-2</td>
<td></td>
<td></td>
</tr>
<tr>
<td>2的30的方-3</td>
<td></td>
<td>42</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>3</td>
<td>y</td>
<td>2的30的方-3</td>
</tr>
<tr>
<td>2</td>
<td>y</td>
<td>42</td>
</tr>
<tr>
<td>1</td>
<td>y</td>
<td>42</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>2的30的方</td>
</tr>
</table>

在现在的情况下，我们在堆上分配了四个事物，但释放了两个。目前，在 2 的 30 次方和 2 的 30 次方 -3 之间还有一些空白还没有被使用。这种情况如何以及为什么发生的具体细节取决于你用什么样的策略来管理堆。不同的程序可以使用不同的内存分配器，有函数库来管理。Rust 程序使用 [jemalloc](http://www.canonware.com/jemalloc/) 达到这个目的。

无论如何，现在回到我们的例子。因为这是堆上的内存，它的生存周期可以超过函数分配 **box** 的范围。然而，在这种情况下，在函数结束后，它不能 **[moving]**。我们需要通过 **main().Box<T>** 释放栈帧，不过，有一个更巧妙的技巧：[Drop](https://doc.rust-lang.org/stable/book/drop.html)。**Drop** 即释放创建 **box** 时分配的内存。太棒了!因此，当删除 **x** 时，它首先释放分配在堆上的内存：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>1</td>
<td>y</td>
<td>42</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>??????</td>
</tr>
<table>

[moving]：我们可以使内存有更长的生存周期，通过转移所有权，有时被称为 **“moving out of the box”**。稍后介绍更复杂的例子。

然后栈帧消失，释放我们所有的内存。

### 参数和引用

我们已经了解了一些栈和堆的基本例子，但是关于函数参数和引用呢?这里有一个小的 Rust 程序：

    fn foo(i: &i32) {
    	let z = 42;
    }
    
    fn main() {
    	let x = 5;
    	let y = &x;
    
    	foo(y);
    }

当我们进入 **main()**，内存看起来像这样：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>1</td>
<td>y</td>
<td>0</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>5</td>
</tr>
<table>

x 值为 5，y 是 x 的一个引用。所以它的值是 x 的内存地址，本例中为0。

调用 foo()，将 y 作为参数传递：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>3</td>
<td>z</td>
<td>42</td>
</tr>
<tr>
<td>2</td>
<td>i</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>y</td>
<td>0</td>
</tr>
<tr>
<td>0</td>
<td>x</td>
<td>5</td>
</tr>
<table>

栈帧不只是本地绑定，它们还可以用作参数。在这种情况下，我们需要有 **i** 作为参数，**z** 作为局部变量绑定。**i** 是参数 **y** 的一个副本。由于 **y** 的值为 **0**，所以 **i** 的值也是 **0**。

这就是为什么引用一个变量不占用任何内存：一个引用的值只是一个指向内存位置的指针。但是如果我们不使用潜在的内存，事情就不能很好的工作了。

### 一个复杂的例子

好吧,让我们一步一步地看下这个复杂的程序：

    fn foo(x: &i32) {
    	let y = 10;
    	let z = &y;
    
    	baz(z);
    	bar(x, z);
    }
    
    fn bar(a: &i32, b: &i32) {
    	let c = 5;
    	let d = Box::new(5);
    	let e = &d;
    
    	baz(e);
    }
    
    fn baz(f: &i32) {
    	let g = 100;
    }
    
    fn main() {
    	let h = 3;
    	let i = Box::new(20);
    	let j = &h;
    
    	foo(j);
    }

首先,我们调用 **main()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

为 **j，i，h** 分配内存。**i** 在堆内，所以它有一个指向那儿的值。

接下来，在 **main()** 结束时调用 **foo()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>5</td>
<td>z</td>
<td>4</td>
</tr>
<tr>
<td>4</td>
<td>y</td>
<td>10</td>
</tr>
<tr>
<td>3</td>
<td>x</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

给 **x，y** 和 **z** 分配空间。参数 **x** 和 **j** 有相同的值，因为这就是我们在其中传递的。它是一个指向地址 **0** 的指针，因为 **j** 指向 **h**。

接下来，**foo()** 调用 **baz()**，传递 **z**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>7</td>
<td>g</td>
<td>100</td>
</tr>
<tr>
<td>6</td>
<td>f</td>
<td>4</td>
</tr>
<tr>
<td>5</td>
<td>z</td>
<td>4</td>
</tr>
<tr>
<td>4</td>
<td>y</td>
<td>10</td>
</tr>
<tr>
<td>3</td>
<td>x</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

给 **f** 和 **g.baz()** 分配内存，这个内存占用空间不大，所以当它结束时，我们可以释放其栈帧：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>5</td>
<td>z</td>
<td>4</td>
</tr>
<tr>
<td>4</td>
<td>y</td>
<td>10</td>
</tr>
<tr>
<td>3</td>
<td>x</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

接下来。**foo()** 调用 **bar()**，参数为 **x** 和 **z**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>2的30次方-1</td>
<td></td>
<td>5</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>10</td>
<td>e</td>
<td>9</td>
</tr>
<tr>
<td>9</td>
<td>d</td>
<td>2的30次方-1</td>
</tr>
<tr>
<td>8</td>
<td>c</td>
<td>5</td>
</tr>
<tr>
<td>7</td>
<td>b</td>
<td>4</td>
</tr>
<tr>
<td>6</td>
<td>a</td>
<td>0</td>
</tr>
<tr>
<td>5</td>
<td>z</td>
<td>4</td>
</tr>
<tr>
<td>4</td>
<td>y</td>
<td>10</td>
</tr>
<tr>
<td>3</td>
<td>x</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

最后，我们堆上分配另一个值，因此我们必须从 **2** 的 **30** 次方减去 **1**。即 **1,073,741,823**。在任何情况下，我们像往常一样设置变量。

**bar()** 结束时,它调用 **baz()**：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>2的30次方-1</td>
<td></td>
<td>5</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>12</td>
<td>g</td>
<td>100</td>
</tr>
<tr>
<td>11</td>
<td>f</td>
<td>4</td>
</tr>
<tr>
<td>10</td>
<td>e</td>
<td>9</td>
</tr>
<tr>
<td>9</td>
<td>d</td>
<td>2的30次方-1</td>
</tr>
<tr>
<td>8</td>
<td>c</td>
<td>5</td>
</tr>
<tr>
<td>7</td>
<td>b</td>
<td>4</td>
</tr>
<tr>
<td>6</td>
<td>a</td>
<td>0</td>
</tr>
<tr>
<td>5</td>
<td>z</td>
<td>4</td>
</tr>
<tr>
<td>4</td>
<td>y</td>
<td>10</td>
</tr>
<tr>
<td>3</td>
<td>x</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

到这里，我们已经到达了最深点! 看看接下来会怎样？

**bar()** 结束后，**f** 和 **g** 就可以除去了：

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>2的30次方-1</td>
<td></td>
<td>5</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>10</td>
<td>e</td>
<td>9</td>
</tr>
<tr>
<td>9</td>
<td>d</td>
<td>2的30次方-1</td>
</tr>
<tr>
<td>8</td>
<td>c</td>
<td>5</td>
</tr>
<tr>
<td>7</td>
<td>b</td>
<td>4</td>
</tr>
<tr>
<td>6</td>
<td>a</td>
<td>0</td>
</tr>
<tr>
<td>5</td>
<td>z</td>
<td>4</td>
</tr>
<tr>
<td>4</td>
<td>y</td>
<td>10</td>
</tr>
<tr>
<td>3</td>
<td>x</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

接下来，我们从 **bar()** 返回。在本例中 **d** 是 **Box<T>** 类型的，那么它也释放了它所指向的：**2** 的 **30** 次方 **-1** 位置的内存。

<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>5</td>
<td>z</td>
<td>4</td>
</tr>
<tr>
<td>4</td>
<td>y</td>
<td>10</td>
</tr>
<tr>
<td>3</td>
<td>x</td>
<td>0</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

之后，**foo()** 返回：
 
<table>
<tr>
<th>地址</th>
<th>名字</th>
<th>值</th>
</tr>
<tr>
<td>2的30次方</td>
<td></td>
<td>20</td>
</tr>
<tr>
<td>...</td>
<td>...</td>
<td>...</td>
</tr>
<tr>
<td>2</td>
<td>j</td>
<td>0</td>
</tr>
<tr>
<td>1</td>
<td>i</td>
<td>2的30次方</td>
</tr>
<tr>
<td>0</td>
<td>h</td>
<td>3</td>
</tr>
<table>

最后是 **main()**，它清除了其余的内存。当 **i** 被 **Drop** 时，它也会清理最后的堆。

### 其他语言怎么做?

大多数语言默认情况下都有一个垃圾收集器 **heap-allocate**。这意味着，每个值都是会被封装。这样做有许多原因，但是它们不在本教程的范围内，我们在这里就不详细说明了。这有一些可能的优化，但也做不到 **100%** 时间的优化。，垃圾收集器能够更好地处理堆内存的问题。

### 使用哪一个?

栈更快，也更容易管理，那么为什么我们还需要堆呢？一个很大原因是，栈分配意味着你只能根据后进先出的语义回收存储。而堆分配严格说来更一般化，允许以任意顺序从池中取出或返回存储器，但是其更复杂，成本更高。

一般来说，更倾向于使用栈分配，因此，Rust 默认情况下是栈分配。栈后进先出模型比较简单，也更基础。一般存在两大影响因素：运行时效率和语义影响。

### 运行时的效率。

栈的内存管理是微不足道的：机器只是增加或减少一个单一的值，即所谓的“栈指针”。堆的内存管理就有点不一般了：堆可以在任意点上分配或释放内存，并且堆上分配的内存块可以是任意大小的，内存管理器的日常工作必然更难，从而能够识别内存以便重用。

如果你想更详细地深入这个主题,[本文](http://www.cs.northwestern.edu/~pdinda/icsclass/doc/dsa.pdf)可以给出一个极好的介绍。

### 语义的影响

栈分配影响 Rust 语言本身，以及开发人员的思维模型。后进先出语义指示 Rust 语言如何自动处理内存管理。如在本章所讨论的那样，甚至一个独特的基于堆的回收箱也可以通过基于栈的后进先出语义驱动。非后进先出语义的灵活性（即表现力）意味着：一般情况下，编译器在编译时无法自动推断应该释放那些内存；它必须依赖于动态协议（可能来自语言本身之外）驱动回收，引用计数器（**Rc<T>** 和 **Arc<T>** 所使用的）就是其中的一个例子。

在往更深层次说呢，逐渐增长的堆分配表达力主要来自有效的运行时支持(如垃圾收集器的形式)和有效的程序员工作(显式的内存管理形式,不需要 Rust 编译器提供验证)。