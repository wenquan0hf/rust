#可变性 

可变性，可以改变东西的能力，与其他语言相比它在 Rust 中有点不同。可变性的第一个方面是它的非默认状态：  

    let x = 5;
    x = 6; // error!

我们可以应用 **mut** 关键字来介绍可变性：
    
    let mut x = 5;
    
    x = 6; // no problem!

这是一个可变的<a href="http://doc.rust-lang.org/stable/book/variable-bindings.html">变量绑定</a>。当一个绑定是可变时，这意味着你可以更改绑定的指向。所以在上面的例子中，不是 **x** 的值发生变化，而是这个绑定从 **i32** 更改为其他。  

如果你想要更改绑定的指向，你将需要一个<a href="http://doc.rust-lang.org/stable/book/references-and-borrowing.html">可变的引用</a>：  

    let mut x = 5;
    let y = &mut x;

**y** 是一个不可变绑定到一个可变的引用，这意味着你不可以将 **y** 绑定到其他 ( **y = &mut z** )，但是你可以改变绑定到 **y**上的东西 ( ***y=5** ).细微的差别。  

当然，如果你两个都需要，可以如下书写：  
    
    let mut x = 5;
    let mut y = &mut x;

现在 **y** 可以绑定到另一个值，并且这个引用的值可以改变。  

注意到 **mut** 是<a href="http://doc.rust-lang.org/stable/book/patterns.html">模式</a>的一部分是非常重要的，你可以像如下代码书写：  

    let (mut x, y) = (5, 6);
    
    fn foo(mut x: i32) {

#内部和外部可变性  

然而，在 Rust 中当我们说有些东西是‘不变’的，这并不意味着它不可以更改：我们说的是它有‘外部可变性’。考虑如下例子，<a href="http://doc.rust-lang.org/stable/std/sync/struct.Arc.html">Arc<T\></a>：  
    
    use std::sync::Arc;
    
    let x = Arc::new(5);
    let y = x.clone();

当我们调用 **clone()** 时，**Arc<T\>** 需要更新引用计数。然而我们在这里还没有使用 **mut**，**x** 是一个不可变绑定，同时我们没有使用 **&mut 5** 或者任何东西。所以谁给？  

要理解这一点，我们需要回到 Rust 的指导哲学的核心，内存安全，和 Rust 保证的机制，<a href="http://doc.rust-lang.org/stable/book/ownership.html">所有权</a>系统，和更具体的，<a href="http://doc.rust-lang.org/stable/book/borrowing.html#The-Rules">借用</a>：  
    
    You may have one or the other of these two kinds of borrows, but not both at the same time:
    
    one or more references (&T) to a resource.
    exactly one mutable reference (&mut T)

所以，这就是‘不变’的真正定义：有两个指针指向内容是否安全？在 **Arc<T\>** 的例子中，是的：变化完全包含在内部结构本身中。它不是面向用户的。为此，将 **&T** 传递给 **clone()**。但是，如果将 **&mut T**  传递给 **clone()**，将会成为一个问题：  

其它类型，诸如 <a href="http://doc.rust-lang.org/stable/std/cell/">std::cell</a> 模块中，具有相反的：内部可变性。例如：  
    
    use std::cell::RefCell;
    
    let x = RefCell::new(42);
    
    let y = x.borrow_mut();

RefCell 用 **borrow_mut()**  函数来将 **&mut** 引用传递到它包含的东西中。那样不危险吗？如果那样做会怎样：  

    use std::cell::RefCell;
    
    let x = RefCell::new(42);
    
    let y = x.borrow_mut();
    let z = x.borrow_mut();

事实上，这将在运行时引起恐慌。这是 **RefCell** 做的事情：它在运行时保证 Rust 的借用规则，同时如果它们违背了规则时的 **panic!**。这允许我们能够绕过 Rust 的不变规则的另一方面。让我们先讲讲吧。  

##字段级可变性

可变性是借用 ( **&mut** ) 或者绑定 ( **let mut** ) 的一个属性。这意味着，例如，你不能有一个<a href="http://doc.rust-lang.org/stable/book/structs.html">结构体</a>既有一些字段可变还有一些不可变：  

    struct Point {
    x: i32,
    mut y: i32, // nope
    }

一个结构体在绑定方面的可变性：  

    struct Point {
    x: i32,
    y: i32,
    }
    
    let mut a = Point { x: 5, y: 6 };
    
    a.x = 10;
    
    let b = Point { x: 5, y: 6};
    
    b.x = 10; // error: cannot assign to immutable field `b.x`

然而，通过使用 **Cell<T\>**，你可以模仿字段级可变性：  

    use std::cell::Cell;
    
    struct Point {
    x: i32,
    y: Cell<i32>,
    }
    
    let point = Point { x: 5, y: Cell::new(6) };
    
    point.y.set(7);
    
    println!("y: {:?}", point.y);

以上代码将打印 **y: Cell { value: 7 }**。我们已经成功更新 **y**。


	
