# While 循环  

Rust 中也有 **while** 循环。如下代码所示：
  
    let mut x = 5; // mut x: i32
    let mut done = false; // mut done: bool
    
    while !done {
    x += x - 3;
    
    println!("{}", x);
    
    if x % 5 == 0 {
    done = true;
    }
    }  

当你不能确定你需要循环多少次的时候 **while** 循环是一个不错的选择。  

如果你需要一个无限循环，你可能尝试如下这样写：  

    while true {  

然而，Rust 有一个专用的关键字， **loop** ，来解决这个问题：  

    loop {

Rust 的控制流分析相对于 **while true**，在处理此构建上有所不同，因为我们知道它将永远循环。一般情况下，我们给编译器的信息越多，编译器越能够更好的处理安全和代码生成问题，所以在你打算实现无限循环时，你应该首选 **loop**。 
 
## 提早结束循环

让我们看看早些时候我们有的 **while** 循环：  

    let mut x = 5;
    let mut done = false;
    
    while !done {
    x += x - 3;
    
    println!("{}", x);
    
    if x % 5 == 0 {
    done = true;
    }
    }
    
我们不得不使用专用的 **mut** 布尔变量绑定， **done**，用来让我们知道什么时候应该退出循环。Rust 有两个关键字来帮助我们修改循环： **break** 和 **continue**。  

在这种情况下，我们可以通过使用 **break** ，来用一种更好的方式书写循环：   

    let mut x = 5;
    
    loop {
    x += x - 3;
    
    println!("{}", x);
    
    if x % 5 == 0 { break; }
    }

现在我们使用 **loop** 来实现无限循环，同时使用 **break** 来提早退出循环。  

**continue** 是相似的，但是它不是结束循环，而是转到下一次循环。以下是只打印奇数的代码：  
    
    for x in 0..10 {
    if x % 2 == 0 { continue; }
    
    println!("{}", x);
    }

**continue** 和 **break** 在 **while** 循环和 <a href="http://doc.rust-lang.org/stable/book/for-loops.html">for循环</a>中都有效。
