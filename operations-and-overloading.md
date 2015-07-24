# 操作符和重载

Rust 允许有限形式的操作符重载。有一些操作符能够被重载。为了支持一个类型之间特定的操作符，有一个你可以实现的特定的特征，然后重载操作符。

例如，可以用 Add 特征重载 + 操作符。

    use std::ops::Add;
    
    #[derive(Debug)]
    struct Point {
    x: i32,
    y: i32,
    }
    
    impl Add for Point {
    type Output = Point;
    
    fn add(self, other: Point) -> Point {
    Point { x: self.x + other.x, y: self.y + other.y }
    }
    }
    
    fn main() {
    let p1 = Point { x: 1, y: 0 };
    let p2 = Point { x: 2, y: 3 };
    
    let p3 = p1 + p2;
    
    println!("{:?}", p3);
    }

在 main 函数中，你可以在两个 Point之间使用 + 操作符，因为我们可以使用 Point 的方法 `Add<Output=Point>`。

有许多操作符可以以这种方式被重载，所有的关联特征都在 `std::ops` 模块中。看看完整列表的文档。　　　　

这些特征的实现遵循一个模式。让我们看看 Add 的更多细节：

    pub trait Add<RHS = Self> {
    type Output;
    
    fn add(self, rhs: RHS) -> Self::Output;
    }

这里总共三种类型包括：impl Add for的类型，默认为 Self 的 RSH，还有 Output。一个表达式 `let z = x + y`，x 是 Self 类型，y 是 RSH 类型，还有 z 是 Self::Output 类型。

这可以让你这样做：

    let p: Point = // ...
    let x: f64 = p + 2i32;