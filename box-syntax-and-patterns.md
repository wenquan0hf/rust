# 盒语法和模式

目前，唯一稳定可靠地方法就是通过 `Box：：new` 方法来创建 `Box`。当然，它不可能在稳定的 Rust 来析构匹配模式下的 `Box`。

不稳定的 `box` 关键字可以用来创建和析构 `Box`。相关的例子如下：    


    #![feature(box_syntax, box_patterns)]
    
    fn main() {
    let b = Some(box 5);
    match b {
    Some(box n) if n < 0 => {
    println!("Box contains negative number {}", n);
    },
    Some(box n) if n >= 0 => {
    println!("Box contains non-negative number {}", n);
    },
    None => {
    println!("No box");
    },
    _ => unreachable!()
    }
    }


注意这个功能目前隐藏在 `box_syntax`(盒创建方法) 和 `box_patterns` （析构和匹配模型）方法，因为这个语法在未来仍可能会被更改。     

## 返回指针

在很多计算机语言中都有指针，用户可以通过返回一个指针来避免返回较大数据结构的拷贝。比如：    


    struct BigStruct {
    one: i32,
    two: i32,
    // etc
    one_hundred: i32,
    }
    
    fn foo(x: Box<BigStruct>) -> Box<BigStruct> {
    Box::new(*x)
    }
    
    fn main() {
    let x = Box::new(BigStruct {
    one: 1,
    two: 2,
    one_hundred: 100,
    });
    
    let y = foo(x);
    }
    
这里面的想法就是通过返回一个盒，用户可以仅仅拷贝一个指针，从而避免 拷贝 `BigStruct` 中的上百个 `int` 数。   

如下为 Rust 的反模式，相反，可以编写成下面的方式：   
 
    #![feature(box_syntax)]
    
    struct BigStruct {
    one: i32,
    two: i32,
    // etc
    one_hundred: i32,
    }
    
    fn foo(x: Box<BigStruct>) -> BigStruct {
    *x
    }
    
    fn main() {
    let x = Box::new(BigStruct {
    one: 1,
    two: 2,
    one_hundred: 100,
    });
    
    let y: Box<BigStruct> = box foo(x);
    }
    
这个方法是一种不牺牲性能的前提下提供了灵活性。   

用户可能会认为这会表现出较差的性能：返回一个值，然后立即用盒装起来？这是最糟糕的模式么？Rust 远远比这些更智能。这并不是将代码进行拷贝。 `main` 为 `box` 分配足够的空间，然后传递一个指针 x 来指向 `foo`。然后，`foo` 将值写回 `Box<T>`。  

下面这一点很重要：指针不仅可以优化代码块中返回的值。也允许调用者来选择他们希望他们如何使用他们的输出。