#所有权

这篇指南是 Rust 已经存在的三个所有权制度之一。这是 Rust 最独特和最令人信服的一个特点，其中 Rust 开发人员应该相当熟悉。所有权即 Rust 如何实现其最大目标和内存安全。这里有几个不同的概念，每一个概念都有它自己的章节：  

- 所有权，即正在读的这篇文章。  

- <a href="http://doc.rust-lang.org/stable/book/references-and-borrowing.html">借用</a>，和与它们相关的功能‘引用’  

- <a href="http://doc.rust-lang.org/stable/book/lifetimes.html">生存期</a>，借用的先进理念 

这三篇文章相关且有序。如果你想完全的理解所有权制度，你将需要这三篇文章。    

#元
  
在我们了解细节之前，这里有关于所有权制度的两个重要说明需要知道。  

Rust 注重安全和速度。它通过许多‘零成本抽象’来完成这些目标，这意味着在 Rust 中，用尽可能少的抽象成本来保证它们正常工作。所有权制度是一个零成本抽象概念的一个主要例子。我们将在这篇指南中提到的所有分析都是在*编译时完成*的。你不用为了任何功能花费任何运行成本。  

然而，这一制度确实需要一定的成本：学习曲线。许多新用户使用我们喜欢称之为‘与借检查器的人斗争’，即 Rust 编译器拒绝编译那些作者认为有效的程序的 Rust 经验。这往往因为程序员对于所有权的怎样工作与 Rust 实现的规则不相符的心理模型而经常出现。你可能在第一次时会经历类似的事情。然而有个好消息：更有经验的 Rust 开发者报告称，一旦他们遵从所有权制度的规则工作一段时间后，他们会越来越少的与借检查器的行为斗争。  

学习了这些后，让我们来了解所有权。  

#所有权 

<a href="http://doc.rust-lang.org/stable/book/variable-bindings.html">变量绑定</a>在 Rust 中有一个属性：它们有它们绑定到的变量的‘所有权’。这意味着当一个绑定超出范围时，将释放它们绑定到资源。例如：  

    fn foo() {
    let v = vec![1, 2, 3];
    }

当 **v** 进入范围时，将新创建新的 <a href="http://doc.rust-lang.org/stable/std/vec/struct.Vec.html">Vec<T\></a>。在这种情况下，向量也在<a href="http://doc.rust-lang.org/stable/book/the-stack-and-the-heap.html">堆</a>上为三个元素分配空间。当 **v** 超出 **foo()**  函数的作用域时，Rust 将清除一切与向量有关的东西，也包括为堆分配的内存。在该作用域结束时，这种情况就一定会发生。  

#移动语义

尽管这里也有很多微妙的东西：Rust 确保*任何给定的资源都有一个确定的绑定*。例如，如果我们有一个向量，我们可以将它赋值给另一个绑定。  

    let v = vec![1, 2, 3];
    
    let v2 = v;
但是，如果我们在之后尝试使用 **v** 时，我们将发现一个错误：  

    let v = vec![1, 2, 3];
    
    let v2 = v;
    
    println!("v[0] is: {}", v[0]);

它会报出如下错误：
    
    error: use of moved value: `v`
    println!("v[0] is: {}", v[0]);
    ^

当我们定义了一个取得所有权的函数，然后在我们已经将它作为参数传递后，然后使用时，类似的情况将会发生：  

    fn take(v: Vec<i32>) {
    // what happens here isn’t important.
    }
    
    let v = vec![1, 2, 3];
    
    take(v);
    
    println!("v[0] is: {}", v[0]);

同样的错误：‘移动值使用’。当我们将所有权转移给其他东西时，我们可以说我们已经‘移动’了我们提到的东西。这里你不需要某种特殊注释，它是 Rust 默认做的事情。   
 
##详细信息

当我们移动一个绑定后，我们不可以使用这个绑定的原因是微妙的，但是很重要。当我们编写如下代码时：  

    let v = vec![1, 2, 3];
    
    let v2 = v;
第一行为向量对象 **v** 和它包含的内容分配内存。向量对象存放在<a href="http://doc.rust-lang.org/stable/book/the-stack-and-the-heap.html">栈</a>中，同时包含一个指向存放在<a href="http://doc.rust-lang.org/stable/book/the-stack-and-the-heap.html">堆</a>中的内容 ( [1, 2, 3] ) 的指针。当我们将 **v** 赋值给 **v2** 时，它将为 **v2** 创建一个这个指针的副本。这意味着将会有两个指针指向堆中的向量内容。它将引进数据竞争，这违反了 Rust 的安全保障。因此，Rust 禁止在我们移动后使用 **v**。  

我们需同样注意某种情况下优化可能删除在栈中字节的真正副本。所以它并不像最初看起来的那样毫无效率。  

##复制类型

在我们将所有权转移到另一个绑定时，我们已经建立了，你不可以使用原来的绑定。然而，这里有一个<a href="http://doc.rust-lang.org/stable/book/traits.html">特性</a>可以改变这种行为，它被称为 **Copy** 。我们还没有讨论过这个特性，但是现在，你可以把它们看作是增加额外行为的一个特殊类型的一个注释。例如：

    let v = 1;
    
    let v2 = v;
    
    println!("v is: {}", v);

在这种情况下，**v** 是一个 **i32**，这实现了 **Copy** 的特性。这意味着，就像一个移动，当我们将 **v** 赋值给 **v2** 时，我们就完成了数据的一个副本。但是，与移动不同，我们在之后仍然可以使用 **v**。这是因为 **i32** 在别处没有指向数据的指针，这是一个完整的副本。  


我们将在<a href="http://doc.rust-lang.org/stable/book/traits.html">特性</a>章节讨论怎样完成你自己类型的复制。  

#不仅仅是所有权

当然，如果我们不得不将每个函数的所有权交回，我们可以写如下代码：

    fn foo(v: Vec<i32>) -> Vec<i32> {
    // do stuff with v
    
    // hand back ownership
    v
    }

这将会特别繁琐。当我们想要取得所有权的东西它将会越糟糕：

    fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // do stuff with v1 and v2
    
    // hand back ownership, and the result of our function
    (v1, v2, 42)
    }
    
    let v1 = vec![1, 2, 3];
    let v2 = vec![1, 2, 3];
    
    let (v1, v2, answer) = foo(v1, v2);

额！返回值类型，返回的行，以及调用的函数获取方式变的更加复杂。  

幸运的是，Rust 提供了一种功能，借用，它能帮助我们解决这个问题。它是下一章节的话题！
