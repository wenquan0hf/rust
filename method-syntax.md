#方法语法

函数很好，但是如果你想要在一些数据上调用很多函数，那是非常不合适的。请思考以下代码：   
    
    baz(bar(foo)));

我们从左往右读这些代码，就会看到 ‘baz bar foo’。但是这并不是我们由内-外调用函数的顺序：‘foo bar baz’。如果我们这样写，会不会更好？  

    foo.bar().baz();

幸运的是，你可能已经猜到了，关于上面问题的答案，可以! Rust 提供了一种可以通过 **impl** 关键字来使用‘方法调用语法’的能力。  

#方法调用 

以下代码展示了它是怎样工作的：  

    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl Circle {
    fn area(&self) -> f64 {
    std::f64::consts::PI * (self.radius * self.radius)
    }
    }
    
    fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());
    }

以上代码将打印 **12.566371**。  

我们已经建了一个表示一个圆的结构体。然后我们写一个 **impl** 块，同时我们在块中定义一个方法，**area**。  

方法使用一个特殊的第一参数，其中有三个变量：**self**，**&self**   和 **&mut self**。你可以将这个第一参数想象成 **foo.bar()** 中的 **foo**。这三种变量对应于 **foo** 可能成为的三种东西：如果它只是堆栈中的一个值使用 **self**，如果它是一个引用使用  **&self**，如果它是一个可变引用使用 **&mut self**。因为我们使用了 **&self**作为 **area** 的参数，我们可以像其他参数一样使用它。由于我们知道它是一个 **Circle**，我们可以像访问其它结构体一样，访问 **radius**。  

我们应默认使用 **&self**，同时相对于取得所有权，你应该更倾向于借用，以及使用不可变的引用来顶替可变的引用。以下是关于所有三个变量的一个例子：  
    
    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl Circle {
    fn reference(&self) {
       println!("taking self by reference!");
    }
    
    fn mutable_reference(&mut self) {
       println!("taking self by mutable reference!");
    }
    
    fn takes_ownership(self) {
       println!("taking ownership of self!");
    }
    }

#链接方法调用

所以，至此我们知道了怎样去调用一个方法，诸如 **foo.bar()**。但是我们原来的例子 **foo.bar().baz()** 的例子怎么办？这就是所谓的‘方法链接’，我们可以通过返回 **self** 来完成它。  

    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl Circle {
    fn area(&self) -> f64 {
    std::f64::consts::PI * (self.radius * self.radius)
    }
    
    fn grow(&self, increment: f64) -> Circle {
    Circle { x: self.x, y: self.y, radius: self.radius + increment }
    }
    }
    
    fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());
    
    let d = c.grow(2.0).area();
    println!("{}", d);
    }

检查返回类型：

    fn grow(&self) -> Circle {

我们只是说我们返回了一个 **Circle**。通过这个方法，我们可以将一个新的圆圈增加到任意大小。  

#关联函数

你还可以定义一个不使用 **self** 作为参数的关联函数。以下代码是在  Rust 代码中非常常见的一种模式：  

    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl Circle {
    fn new(x: f64, y: f64, radius: f64) -> Circle {
    Circle {
    x: x,
    y: y,
    radius: radius,
    }
    }
    }
    
    fn main() {
    let c = Circle::new(0.0, 0.0, 2.0);
    } #

这个‘关联函数’为我们生成一个新的 **Circle**。请注意，关联函数被  **Struct::function()** 语法调用，而不是 **ref.method()** 语法。其他一些语言称关联函数为‘静态方法’。  

#生成器模式

假如我们想要我们的用户能够创建 Circles，但是我们只允许他们设置他们关心的属性。否则，**x** 和 **y** 的属性将会是 **0.0**，同时 **radius** 为 **1.0**。Rust 没有方法重载，参数命名或者变量参数。我们使用生成器模式来代替它。如以下代码所示：  

    struct Circle {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl Circle {
    fn area(&self) -> f64 {
    std::f64::consts::PI * (self.radius * self.radius)
    }
    }
    
    struct CircleBuilder {
    x: f64,
    y: f64,
    radius: f64,
    }
    
    impl CircleBuilder {
    fn new() -> CircleBuilder {
    CircleBuilder { x: 0.0, y: 0.0, radius: 1.0, }
    }
    
    fn x(&mut self, coordinate: f64) -> &mut CircleBuilder {
    self.x = coordinate;
    self
    }
    
    fn y(&mut self, coordinate: f64) -> &mut CircleBuilder {
    self.y = coordinate;
    self
    }
    
    fn radius(&mut self, radius: f64) -> &mut CircleBuilder {
    self.radius = radius;
    self
    }
    
    fn finalize(&self) -> Circle {
    Circle { x: self.x, y: self.y, radius: self.radius }
    }
    }
    
    fn main() {
    let c = CircleBuilder::new()
    .x(1.0)
    .y(2.0)
    .radius(2.0)
    .finalize();
    
    println!("area: {}", c.area());
    println!("x: {}", c.x);
    println!("y: {}", c.y);
    }

我们这里所做的就是建立了另一个结构体，**CircleBuilder**。我们已经对它定义了我们的生成器方法。我们也已经在 **Circle** 上定义了我们的 **area()** 方法。我们又在 **CircleBuilder** 上定义了另一方法：**finalize()**。这个方法从生成器中创建了我们的最后的  **Circle**。现在，我们已经使用类型系统来强化我们关心的事情：我们可以使用在 **CircleBuilder** 上的方法来限制以我们选择的任何方式制作 **Circle**。
