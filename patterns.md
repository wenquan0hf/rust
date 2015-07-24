# 模式

模式在 Rust 中颇为常见。我们在[变量绑定](http://doc.rust-lang.org/stable/book/variable-bindings.html)，[match语句](http://doc.rust-lang.org/stable/book/match.html)和其他地方也使用它们。让我们继续旋风般的学习模式可以做的所有事情!  
  
快速学习：你可以直接匹配文字，同时 **\_** 充当一个‘任何’的事件：  

    let x = 1;
    
    match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
    }

以上代码将打印 **one**。  

## 多个模式 

你可以使用 **|** 来匹配多个模式：  

    let x = 1;
    
    match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
    }

以上代码将打印 **one or two**。  

## 范围 

你可以使用 **...** 来匹配值的范围：  

    let x = 1;
    
    match x {
    1 ... 5 => println!("one through five"),
    _ => println!("anything"),
    }

以上代码将打印 **one through five**。  

范围通常在整数和**字符**的情况下使用：  

    let x = '💅';
    
    match x {
    'a' ... 'j' => println!("early letter"),
    'k' ... 'z' => println!("late letter"),
    _ => println!("something else"),
    }

以上代码将打印 **something else**。  

## 绑定

你可以使用 **@** 将值绑定到名称：  

    let x = 1;
    
    match x {
    e @ 1 ... 5 => println!("got a range element {}", e),
    _ => println!("anything"),
    }

以上代码将打印 **got a range element 1**。当你想要操作数据结构中的一部分的一个复杂匹配时，这将非常有用：  

    #[derive(Debug)]
    struct Person {
    name: Option<String>,
    }
    
    let name = "Steve".to_string();
    let mut x: Option<Person> = Some(Person { name: Some(name) });
    match x {
    Some(Person { name: ref a @ Some(_), .. }) => println!("{:?}", a),
    _ => {}
    }

以上代码将打印 **Some("Steve")**：我们已经把内部**名称**绑定到 **a**。  

如果你使用 **@** 和 **|** 时，你需要确保在模式的每个部分都已经绑定好名称。  

    let x = 5;
    
    match x {
    e @ 1 ... 5 | e @ 8 ... 10 => println!("got a range element {}", e),
    _ => println!("anything"),
    }

## 忽略变量

如果你要匹配一个包含变量的枚举，你可以使用 **..** 来忽略变量的值和类型：  

    enum OptionalInt {
    Value(i32),
    Missing,
    }
    
    let x = OptionalInt::Value(5);
    
    match x {
    OptionalInt::Value(..) => println!("Got an int!"),
    OptionalInt::Missing => println!("No such luck."),
    }

以上代码将输出 **Got an int！**。  

## 守卫 

你可以通过 **if** 语句来介绍‘守卫匹配’：  

    enum OptionalInt {
    Value(i32),
    Missing,
    }
    
    let x = OptionalInt::Value(5);
    
    match x {
    OptionalInt::Value(i) if i > 5 => println!("Got an int bigger than five!"),
    OptionalInt::Value(..) => println!("Got an int!"),
    OptionalInt::Missing => println!("No such luck."),
    }

以上代码将输出 **Got an int！**。  

## ref 和 ref mut

如果你想要获得一个[引用](http://doc.rust-lang.org/stable/book/references-and-borrowing.html)，可以使用  **ref** 关键字：  

    let x = 5;
    
    match x {
    ref r => println!("Got a reference to {}", r),
    }

以上代码将打印出 **Got a reference to 5**。  

在这里，在 **match** 中的 **r** 的类型为 **&i32**。换句话说，在模式中，使用 **ref** 关键字*创建*一个引用以供使用。如果你需要一个可变引用，**ref mut** 将会以相同的方式工作：  

    let mut x = 5;
    
    match x {
    ref mut mr => println!("Got a mutable reference to {}", mr),
    }

## 重构

如果你有一个复合数据类型，诸如[结构体](http://doc.rust-lang.org/stable/book/structs.html)，你可以在一个模式中重构它：  

    struct Point {
    x: i32,
    y: i32,
    }
    
    let origin = Point { x: 0, y: 0 };
    
    match origin {
    Point { x: x, y: y } => println!("({},{})", x, y),
    }

如果我们只关心一部分值，我们不需要给出它们所有的名字：  
    
    struct Point {
    x: i32,
    y: i32,
    }
    
    let origin = Point { x: 0, y: 0 };
    
    match origin {
    Point { x: x, .. } => println!("x is {}", x),
    }

以上代码将打印出 **x is 0**。  

你可以在任何成员中做这种匹配，而不仅仅是第一个成员：  

    struct Point {
    x: i32,
    y: i32,
    }
    
    let origin = Point { x: 0, y: 0 };
    
    match origin {
    Point { y: y, .. } => println!("y is {}", y),
    }

以上代码将打印出 **y is 0**。  

这种‘重构’行为对于任何复合数据类型都有效，比如数组或者枚举。  

## 混合与匹配

这里有很多种不同的方法来匹配东西，它们又可以被混合和匹配，完全取决于你做什么任务：  

    match x {
    Foo { x: Some(ref name), y: None } => ...
    }

模式非常强大。充分利用它。
	
