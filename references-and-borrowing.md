#引用与借用

这篇指南是 Rust 已经存在的三个所有权制度之一。这是 Rust 最独特和最令人信服的一个特点，其中 Rust 开发人员应该相当熟悉。所有权即 Rust 如何实现其最大目标和内存安全。这里有几个不同的概念，每一个概念都有它自己的章节：  

- <a href="http://doc.rust-lang.org/stable/book/ownership.html">所有权</a>，即正在读的这篇文章。  

- 借用，和与它们相关的功能‘引用’  

- <a href="http://doc.rust-lang.org/stable/book/lifetimes.html">生存期</a>，借用的先进理念 

这三篇文章相关且有序。如果你想完全的理解所有权制度，你将需要这三篇文章。    

#元
  
在我们了解细节之前，这里有关于所有权制度的两个重要说明需要知道。  

Rust 注重安全和速度。它通过许多‘零成本抽象’来完成这些目标，这意味着在 Rust 中，用尽可能少的抽象成本来保证它们正常工作。所有权制度是一个零成本抽象概念的一个主要例子。我们将在这篇指南中提到的所有分析都是在*编译时完成*的。你不用为了任何功能花费任何运行成本。  

然而，这一制度确实需要一定的成本：学习曲线。许多新用户使用我们喜欢称之为‘与借检查器的人斗争’，即 Rust 编译器拒绝编译那些作者认为有效的程序的 Rust 经验。这往往因为程序员对于所有权的怎样工作与 Rust 实现的规则不相符的心理模型而经常出现。你可能在第一次时会经历类似的事情。然而有个好消息：更有经验的 Rust 开发者报告称，一旦他们遵从所有权制度的规则工作一段时间后，他们会越来越少的与借检查器的行为斗争。  

学习了这些后，让我们来了解借用。

#借用

在<a href="http://doc.rust-lang.org/stable/book/ownership.html">所有权</a>章节的末尾部分，我们有一个令人讨厌的功能，如下所示：

    fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // do stuff with v1 and v2
    
    // hand back ownership, and the result of our function
    (v1, v2, 42)
    }
    
    let v1 = vec![1, 2, 3];
    let v2 = vec![1, 2, 3];
    
    let (v1, v2, answer) = foo(v1, v2);

这不是惯用的 Rust，然而，由于它不能利用借出，以下是第一步：  

    fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    // do stuff with v1 and v2
    
    // return the answer
    42
    }
    
    let v1 = vec![1, 2, 3];
    let v2 = vec![1, 2, 3];
    
    let answer = foo(&v1, &v2);
    
    // we can use v1 and v2 here!

我们使用了一个引用：**&Vec<i32\>**，而不是将 **Vec<i32\>**  作为我们的参数。同时，我们不是直接传递  **V1**  和  **V2**，我们传递 **&V1** 和 **&V2**。我们称 **&T**  为一个‘引用’，它借用了所有权，而不是拥有了资源。借用东西的绑定当它超出作用域时，它不解除分配给它的资源。这意味着在调用 **foo()** 函数后，我们原始的绑定可以再次被使用。  

引用与绑定类似，它们都是不可变的。这意味着，在 **foo()** 函数中，向量根本不能改变：  

    fn foo(v: &Vec<i32>) {
     v.push(5);
    }
    
    let v = vec![];
    
    foo(&v);

错误：  

    error: cannot borrow immutable borrowed content `*v` as mutable
    v.push(5);
    ^

推入一个值改变了这个变量，所以我们不允许这样做。  

#&mut引用  

这里有第二种引用：**&mut T**。一个‘可变引用’允许你改变你借用的资源。例如：  

    let mut x = 5;
    {
    let y = &mut x;
    *y += 1;
    }
    println!("{}", x);

以上代码将输出 **6**。我们将 **y** 标记为 **x** 的一个可变引用，然后将 **y** 指向的内容加 **1**。你将会注意到 **x** 也不得不被标记为 **mut**，如果不被标记，我们不能将一个可变值借用给一个不可变值。  
 
其他方面，**&mut** 引用与其他引用一样。尽管它们之间以及它们如何相互作用有*一个*很大的区别。你可以在上面的例子中看到一些可疑的东西，因为我们需要那个用 **{** 和 **}** 包围的额外空间。如果我们删除它们，我们将会得到一个错误：  

    error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
       ^
    note: previous borrow of `x` occurs here; the mutable borrow prevents
    subsequent moves, borrows, or modification of `x` until the borrow ends
    let y = &mut x;
     ^
    note: previous borrow ends here
    fn main() {
    
    }
    ^

事实证明，这里是有规则的。  
  

#规则  

这里有关于借用在 Rust 中的规则：  

首先，任何借用必须持续比所有者的作用域小。其次，你可能有一个或者两种其他的借用，但是两种不能在同一时间同时使用：  

- 一个资源的从 0 到 N 的引用 ( **&T** ）。

- 一个可变的引用 ( **&mut T** ）  

你可能会注意到这与数据竞争的定义非常相似，尽管并不是完全相同:  

    There is a ‘data race’ when two or more pointers access the same memory location at the same time, where at least one of them is writing, and the operations are not synchronized.

使用引用，由于它们都不编写，所以你喜欢用多少用多少。如果你想要编写，你需要两个或者更多的指针指向同一内存，你每次仅仅可以使用一个 **&mut**。这就是 Rust 如何在编译时避免数据竞争：如果我们违反了规则，我们将会得到错误。  

学习了以上内容后，让我们再次重新考虑我们的例子。  

##在作用域中的思考  

以下为相关代码：  

    let mut x = 5;
    let y = &mut x;
    
    *y += 1;
    
    println!("{}", x);

这些代码给出我们如下错误：  

    error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
       ^

这是因为我们违反了规则，我们有一个 **&mut T** 指向 **x** ，所以我们不允许创建任何 **&T**。反之亦然。以下注释暗示我们如何思考这个问题：  

    note: previous borrow ends here
    fn main() {
    
    }
    ^

换句话说，这个可变的借用贯穿了我们剩余的示例。我们想要的是在我们尝试调用  **printLn!**  之*前*，可变借用结束，然后使用不变的借用。在 Rust 中，借用被绑定在对于借用有效的作用域内。我们的作用域如下所示：  

    let mut x = 5;
    
    let y = &mut x;// -+ &mut borrow of x starts here
       //  |
    *y += 1;   //  |
       //  |
    println!("{}", x); // -+ - try to borrow x here
       // -+ &mut borrow of x ends here

作用域冲突：我们不能让 **&x** 和 **y** 在一个作用域内。  

因此我们添加大括号：  

      let mut x = 5;
    
    {   
    let y = &mut x; // -+ &mut borrow starts here
    *y += 1;//  |
    }   // -+ ... and ends here
    
    println!("{}", x);  // <- try to borrow x here

此时没有问题。我们可变的借用在我们创建不变的借用之前，超出了作用域。但是作用域是我们可以观察到一个借用可以持续多久的关键。  

##借用问题预防

为什么会有这些预防性的规则？正如我们所指出的，这些规则预防了数据竞争现象。什么样的问题是引起数据竞争现象的原因呢？以下有几个原因。  

###迭代器失效 

其中一个例子是当你尝试改变你正在循环的集合时将会出现的‘迭代器失效’。Rust 的借用检查器防止这种情况的发生：  
    
    let mut v = vec![1, 2, 3];
    
    for i in &v {
    println!("{}", i);
    }

以上代码将打印出从 1 到 3 的数字。当我们循环访问这些向量时，我们仅仅给出了这些元素的引用。**V**  本身作为不可变的借用，这意味着，在我们遍历的过程中我们不能改变它：  

    let mut v = vec![1, 2, 3];
    
    for i in &v {
    println!("{}", i);
    v.push(34);
    }

以下是出现的错误：  

    error: cannot borrow `v` as mutable because it is also borrowed as immutable
    v.push(34);
    ^
    note: previous borrow of `v` occurs here; the immutable borrow prevents
    subsequent moves or mutable borrows of `v` until the borrow ends
    for i in &v {
      ^
    note: previous borrow ends here
    for i in &v {
    println!(“{}”, i);
    v.push(34);
    }
    ^

我们不能修改 **V**，因为它在循环中被借用。  

###释放内存后再使用 

引用必须与它们引用到的资源存活时间一样长。 Rust 会检查你的引用的作用域来保证它为真。  

如果 Rust 不检查这个属性，我们可能会无意间使用一个无效的引用。例如：  

    let y: &i32;
    { 
    let x = 5;
    y = &x;
    }
    
    println!("{}", y);

我们将会得到以下错误：

    error: `x` does not live long enough
    y = &x;
     ^
    note: reference must be valid for the block suffix following statement 0 at
    2:16...
    let y: &i32;
    { 
    let x = 5;
    y = &x;
    }
    
    note: ...but borrowed value is only valid for the block suffix following
    statement 0 at 4:18
    let x = 5;
    y = &x;
    }

换句话说，**y** 只在 **X** 存在的作用域内有效。一旦 **x** 消失了，它将会变成一个 **x** 的无效引用。因此，上面代码中的错误中说借用‘活的时间不够长’，因为它在有效的矢量的时间内是无效的。  

当在引用到的变量之*前*声明引用时，会发生同样的问题：  
    
    let y: &i32;
    let x = 5;
    y = &x;
    
    println!("{}", y);

我们会得到以下错误：  

    error: `x` does not live long enough
    y = &x;
     ^
    note: reference must be valid for the block suffix following statement 0 at
    2:16...
    let y: &i32;
    let x = 5;
    y = &x;
    
    println!("{}", y);
    }
    
    note: ...but borrowed value is only valid for the block suffix following
    statement 1 at 3:14
    let x = 5;
    y = &x;
    
    println!("{}", y);
    }