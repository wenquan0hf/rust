##属性

在 Rus t语言中，声明可以用 ‘attributes’ 来注释。它们看起来像下面这样

    #[test]

或者是像这样:

    #![test]

两者的区别是!,！改变了属性所能够适用的事物:

    #[foo]
    struct Foo;
    
    mod bar {
    #![bar]
    }


`#(foo)` 属性应用到下一个项目,而这个项目就是结构体声明。 #![bar] 属性适用于包含它的项目,这种属性是一个 mod 声明。否则,它们是相同的。这样都在某种程度上改变项了目的意义。

例如,考虑这样一个函数:

    #[test]
    fn check() {
    assert_eq!(2, 1 + 1);
    }

这是用 #[test] 来标志的。这意味着它是特殊的:当您运行测试时,该函数将执行。当你和往常一样编译时, #[test] 甚至不会被包括在编译的范围之内。这个函数是现在一个测试函数。

属性也可能含有额外的数据:
    
    #[inline(always)]
    fn super_fast_fn() {
    或者甚至是关键字和值:
    #[cfg(target_os = "macos")]
    mod macos_only {

Rust 属性被用于许多不同的事情。这有一个属性的完整[引用列表](http://doc.rust-lang.org/stable/reference.html#attributes)。目前,不允许用户创建自己的属性,只能由 Rust 编译器来定义它们。
 

