# 关联类型

关联类型是 Rust 的类型系统一个强大的部分。它们与一个“类型家族”的概念有关，换句话说，就是将多种类型组合在一起。这样描述有点抽象，所以让我们深入理解一个例子。如果你想写一个特征，名字为 Graph ，你有2种类型是通用的：节点类型和边类型。所以你可以写一个特征 ，`Graph<N, E>`，看起来像这样：

    trait Graph<N, E> {
    fn has_edge(&self, &N, &N) -> bool;
    fn edges(&self, &N) -> Vec<E>;
    // etc
    }

但是这类代码结束的不太合适。例如，任何想要以 Graph 为参数的函数，现在也需要在节点和边类型上是通用的：

    fn distance<N, E, G: Graph<N, E>>(graph: &G, start: &N, end: &N) -> u32 { ... }

我们的距离计算与我们的类型 Edge 无关，所以填充在声明中的 E 只是一个无关变量。

我们真正想说的是,边 Edge 和 Node 类型一起构成 Graph 类。我们可以称它们为关联类型：

    trait Graph {
    type N;
    type E;
    
    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
    // etc
    }

现在，我们的客户端可以抽象出一个给定的 Graph：

    fn distance<G: Graph>(graph: &G, start: &G::N, end: &G::N) -> uint { ... }

这里不需要处理 Edge 类型！

让我们更详细地去学习这些知识。

## 定义关联类型

让我们构建特征 Graph。这里是它的定义：

    trait Graph {
    type N;
    type E;
    
    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
    }

这很简单。关联类型要在特征函数体内使用 type 关键字。　　　　

这些 type 声明可以有和函数一样的功能。例如，如果我们希望我们的 N 类型实现 Display，所以我们可以打印节点，我们可以这样做：

    use std::fmt;
    
    trait Graph {
    type N: fmt::Display;
    type E;
    
    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
    }
    
## 关联类型的实现

就像任何特征，使用关联类型的特征要使用 impl 关键字提供实现。这是一个Graph 的简单实现：

    struct Node;
    
    struct Edge;
    
    struct MyGraph;
    
    impl Graph for MyGraph {
    type N = Node;
    type E = Edge;
    
    fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
    true
    }
    
    fn edges(&self, n: &Node) -> Vec<Edge> {
    Vec::new()
    }
    }

这个简单的实现总是返回 true 和一个空的 `Vec <Edge>`，但它给了你一个如何实现这种功能的办法。我们首先需要三个struct，一个图，一个节点，一个边。如果使用不同的类型，那也行，我们只是要用所有这三个变量的 struct。　　　　

接下来是 impl，它就像实现任何其他特征一样。　　　　

在这里,我们使用 = 定义我们的关联类型。特征使用的名字放在 = 的左侧，我们实现的具体类型放在 = 右边。最后，我们使用函数中声明的具体类型。

## 关联类型的特征对象

还有一个语法我们应该谈论一下：特征对象。如果你想创建一个关联类型的特征对象，如下：

    let graph = MyGraph;
    let obj = Box::new(graph) as Box<Graph>;

你会得到两个错误：

    error: the value of the associated type `E` (from the trait `main::Graph`) must
    be specified [E0191]
    let obj = Box::new(graph) as Box<Graph>;
      ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    24:44 error: the value of the associated type `N` (from the trait
    `main::Graph`) must be specified [E0191]
    let obj = Box::new(graph) as Box<Graph>;
      ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我们不能像这样创建一个特征对象，因为我们不知道关联类型。相反，我们可以写：

    let graph = MyGraph;
    let obj = Box::new(graph) as Box<Graph<N=Node, E=Edge>>;

N=Node 语法允许我们为 N 类型参数提供一个具体的类型，Node。E=Edge 也一样。如果我们没有提供这个约束，我们无法确定哪一个 impl 与这个特征对象相匹配。