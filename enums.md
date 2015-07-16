#枚举 

在 Rust 中**枚举**是一个类型，它表示可能是几个可能变量中的一个数据:  

    enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
    }

每个变量可以选择性的有与之关联的数据。定义变量的语法与之前定义结构体的语法类似：你可以有不包含数据的变量（如 unit-like 结构体），已经命名数据的变量，和未命名数据的变量（如数据结构体）。然而，不同于单独的结构体定义，**枚举**是一种类型。枚举的值可以与任何变量匹配。为此，一个枚举有时被称为‘总和类型’：这个枚举的可能值的集合是每个变量的可能值的集合的总和。    

我们使用 :: 语法来使用每个变量的名称：它们由**枚举**本身的名字来划分作用域。这允许以下的代码实现:
    
    let x: Message = Message::Move { x: 3, y: 4 };
    
    enum BoardGameTurn {
    Move { squares: i32 },
    Pass,
    }
    
    let y: BoardGameTurn = BoardGameTurn::Move { squares: 1 }; 

这两个变量都被命名为 **Move**，但是由于它们局限于枚举的名称，它们可以没有冲突的被使用。  

一个枚举类型的值包含它是哪个变量的信息，不包括与该变量相关联的任何数据。它有时被称为‘标签结合’，因为数据中包括一个‘标签’用来指示数据的类型。编译器使用这些信息来保证你访问枚举中的数据的安全性。例如，你不能简单的就好像它是可能变量之一一样来尝试解构一个值：

    fn process_color_change(msg: Message) {
    let Message::ChangeColor(r, g, b) = msg; // compile-time error
    }
  
不支持这些操作可能会看起来相当受限制，但是这是一个我们可以克服的限制。这里有两种方式：通过我们自己实现相等，或者通过模式匹配带有<a href="http://doc.rust-lang.org/stable/book/match.html">模式</a>表达式的变量，你将在下一节中学习这些内容。我们对于 Rust 怎样实现平等了解还不够，但是我们会在<a href="http://doc.rust-lang.org/stable/book/traits.html">特性</a>章节学习到。
