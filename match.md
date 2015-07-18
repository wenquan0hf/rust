#匹配

通常情况下，一个简单的 <a href="http://doc.rust-lang.org/stable/book/if.html">if</a>/**else** 是不足够的，因为你可能有两个以上的选择。此外，条件可能变得相当复杂。Rust 有关键字      **match**，允许你用更强大的 match 关键字，取代复杂的 **if/else** 集合。如下所示：  

    let x = 5;
    
    match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    4 => println!("four"),
    5 => println!("five"),
    _ => println!("something else"),
    }

**match** 采用表达式的形式，然后根据它的值来分支。分支的每个‘臂’都是 **val=>expression** 的形式。当值匹配时，这个臂的表达式将被执行实现。之所以称之为 match 是因为‘模式匹配’的术语，而这种正是  **match** 实现的形式。这里有一整章关于<a href="http://doc.rust-lang.org/stable/book/patterns.html">模式</a>，包含了这里可能的所有模式。  

那么大的优势是什么呢？这里有几个优势。首先，匹配保障‘穷尽性检查’。你是否看到了最后一个下划线 (\_) ？如果我们删除该下划线，Rust 将会给出我们如下错误：  
    
    error: non-exhaustive patterns: `_` not covered

换句话说，Rust 试图告诉我们，我们忘记了一个值。因为 **x** 是一个整数，Rust 知道它可以有许多不同的值 - 例如，**6**。然而，如果没有 **\_**，将没有可匹配项，所以 Rust 拒绝编译这段代码。**\_**  就像一个‘全匹配通配符’。如果其他的匹配项都不能匹配，将会匹配 **\_** 的分支，由于我们有了全匹配通配符，我们现在对于 **x** 的每个可能值都有一个匹配项，所以我们的程序将会被成功编译。  

**match** 也是一个表达式，这意味着我们可以直接在一个 **let** 绑定的右边使用它，或者直接作为表达式使用：  

    let x = 5;
    
    let number = match x {
    1 => "one",
    2 => "two",
    3 => "three",
    4 => "four",
    5 => "five",
    _ => "something else",
    };

有时它是将某种东西从一种类型转换为另一种类型的好方式。  

#枚举匹配

**match** 关键字的另一个重要的用途是处理一个枚举的可能变量：  
    
    enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
    }
    
    fn quit() { /* ... */ }
    fn change_color(r: i32, g: i32, b: i32) { /* ... */ }
    fn move_cursor(x: i32, y: i32) { /* ... */ }
    
    fn process_message(msg: Message) {
    match msg {
    Message::Quit => quit(),
    Message::ChangeColor(r, g, b) => change_color(r, g, b),
    Message::Move { x: x, y: y } => move_cursor(x, y),
    Message::Write(s) => println!("{}", s),
    };
    }

再次，Rust 编译器检查内容详尽，所以它要求你对于枚举的每个变量都有一个匹配的臂。如果你遗漏了一个，它将会给你一个编译时错误，除非你使用 **\_**。   

与之前使用的 **match** 不同，你不可以使用正常的 **if** 语句来做到这一点。你可以使用可以被看做 **match** 的一种缩写形式的<a href="http://doc.rust-lang.org/stable/book/if-let.html">if let</a> 语句。
