##通用函数调用语法

有时,函数可以有相同的名字。看看下面这段代码:
    
    trait Foo {
    fn f(&self);
    }
    
    trait Bar {
    fn f(&self);
    }
    
    struct Baz;
    
    impl Foo for Baz {
    fn f(&self) { println!("Baz’s impl of Foo"); }
    }
    
    impl Bar for Baz {
    fn f(&self) { println!("Baz’s impl of Bar"); }
    }
    
    let b = Baz;

如果我们试图调用 b.f(),我们就会得到一个错误:

    error: multiple applicable methods in scope [E0034]
    b.f();
      ^~~
    note: candidate #1 is defined in an impl of the trait `main::Foo` for the type
    `main::Baz`
    fn f(&self) { println!("Baz’s impl of Foo"); }
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    note: candidate #2 is defined in an impl of the trait `main::Bar` for the type
    `main::Baz`
    fn f(&self) { println!("Baz’s impl of Bar"); }
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我们需要一种消除歧义的方法。这种方法称为“通用函数调用语法”,它这种语法看起来是下面这样的:

    Foo::f(&b);
    Bar::f(&b);
    我们让它停止一下。
    Foo::
    Bar::
    
这些部分调用的类型有两个特征:Foo 和 Bar。这最终实际上是做了两者之间的消歧:Rust 从两者中调用你所使用的特征名称。

    f(&b)

当我们用 [method syntax](http://doc.rust-lang.org/stable/book/method-syntax.html)（）调用一个方法比如 b.f(),如果 f() 含有 &self，Rust 会自动 borrow b。在本例这种情况下,Rust 不会,所以我们需要传递一个具体的 &b。

###尖括号形式

现在我们谈论一下 UFCS 形式:

    Trait::method(args);

这只是一种速记收手法。下面是在某些情况下需要使用的扩展形式:

    <Type as Trait>::method(args); 

< >::语法是一种提供类型提示的方法。类型写在 < >s 里面。在本例中,类型是Type as Trait,表明我们希望特征的方法在这里被调用。在不出现歧义的情况下，特征部分是可选的。这同样适用于用尖括号括,因此要用较短的形式。

这是使用更长形式的一个例子:

    trait Foo {
    fn clone(&self);
    }
    
    #[derive(Clone)]
    struct Bar;
    
    impl Foo for Bar {
    fn clone(&self) {
    println!("Making a clone of Bar");
    
    <Bar as Clone>::clone(self);
    }
    }
    
这将调用 Clone 特征的 Clone () 方法,而不是 Foo 特征的方法。

