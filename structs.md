# 结构体

结构体是创建更复杂的数据类型的一种方式。例如，如果我们做涉及在二维空间中的坐标计算时，我们可能既需要 **x** 的值，也需要 **y** 的值：  

    let origin_x = 0;
    let origin_y = 0;

一个结构体让我们将二者结合成为一个单一的，统一的数据类型：  
    
    struct Point {
    x: i32,
    y: i32,
    }
    
    fn main() {
    let origin = Point { x: 0, y: 0 }; // origin: Point
    
    println!("The origin is at ({}, {})", origin.x, origin.y);
    }
    
这里有许多东西需要讲，所以先在这里打断一下。我们用一个 **struct** 关键字来声明一个**结构体** ，同时 struct 后跟着此结构体的名字。按照惯例，**结构体**以大写字母开头，同时都为驼峰式书写：**PointInSpace** ，而不是**Point\_In_Space**。  

我们可以通过 **let** 来创建一个我们的结构体的实例，像往常一样，但是我们使用一个**关键字：值**的语法风格来设置每个字段。不要求顺序与原始声明的顺序相同。  

最后，因为字段需要名称，我们可以通过点符号来访问字段：**origin.x**。  

在默认情况下，结构体中的值像 Rust 中的其他绑定一样都是不可变的。我们使用 **mut** 关键字来使它们为可变：  

    struct Point {
    x: i32,
    y: i32,
    }
    
    fn main() {
    let mut point = Point { x: 0, y: 0 };
    
    point.x = 5;
    
    println!("The point is at ({}, {})", point.x, point.y);
    }

以上代码将打印 **The point is at (5, 0)**。

Rust 在语言层面不支持字段可变性，所以你不能像以下这样书写代码：   

    struct Point {
    mut x: i32,
    y: i32,
    }

可变性是绑定的一个属性，而不是结构体本身。如果你习惯于字段级别可变性，第一次用起来会感觉很奇怪，但是它极大的简化了一些东西。它甚至允许您使某东西为可变，但是仅仅适用于短时间内：  

    struct Point {
    x: i32,
    y: i32,
    }
    
    fn main() {
    let mut point = Point { x: 0, y: 0 };
    
    point.x = 5;
    
    let point = point; // this new binding can’t change now
    
    point.y = 6; // this causes an error
    }

## 语法更新

一个**结构体**可以包含 **..** 来表明你想要使用一些其他结构体的副本用于某些值。例如：  

    struct Point3d {
    x: i32,
    y: i32,
    z: i32,
    }
    
    let mut point = Point3d { x: 0, y: 0, z: 0 };
    point = Point3d { y: 1, .. point };
    
以上代码中给了 **point** 结构体一个新的 **y**，但是仍然保持 **x** 和 **z** 的原来的值。它也并不一定是相同的**结构体**，当你想要添加新的变量时可以使用这个语法，同时它会复制你没有指定的值:  

    let origin = Point3d { x: 0, y: 0, z: 0 };
    let point = Point3d { z: 1, x: 2, .. origin };

## 数组结构体

Rust 有另外一个数据类型就像一个[数组](http://doc.rust-lang.org/stable/book/primitive-types.html#tuples)与一个结构体的混合，称为“数组结构体”。数组结构体有一个名字，但是它们的字段没有名字：

    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);

以上代码中的两个结构体不相等，即使它们有相同的值：

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);

与数组结构体相比，使用结构体几乎总是比较好。我们可以像如下代码一样书写 **Color** 和 **Point**：

    struct Color {
    red: i32,
    blue: i32,
    green: i32,
    }
    
    struct Point {
    x: i32,
    y: i32,
    z: i32,
    }

现在，我们有真正的名字，而不是位置。好名字固然重要，同时，结构体我们需要有真正的名字。

在*一种*情况下，数组结构体非常有用，尽管，那样一个数组结构体仅包含一个元素。我们称之为‘新型’模式，因为它允许我们创建一个新的类型，不同于其包含的值，并表示它自己的语义：
    
    struct Inches(i32);
    
    let length = Inches(10);
    
    let Inches(integer_length) = length;
    println!("length is {} inches", integer_length);

正如你所看到的，你可以通过一个非结构化的 **let** 关键字来提取内在的整数类型，正如通常的数组一样。在这种情况下，**let Inches(integer_length)** 将 **10** 赋值给 **integer\_length**。

## Unit-like 结构体

你可以定义一个根本没有任何成员的结构体：  

    struct Electron;

这样的结构体被称为 ‘unit-like’ ，因为它类似于空数组，**()**，有时被称为 ‘unit’。与数组结构体一样，它定义了一个新的类型。  

这种结构体通常对自己没有什么用处（虽然它有时可以作为一个标记类型使用），但是结合其他的功能，它可以变的有用。例如，一个库可能想要要求你创建一个可以实现某些特定[特征](http://doc.rust-lang.org/stable/book/traits.html)的结构体来处理事件。如果你没有需要在结构体中存储的数据，你可以只创建一个 unit-like 结构体。
