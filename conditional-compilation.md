## 条件编译

Rust 有一个特殊属性 **#[cfg]**，它允许你编译基于标志的代码并传递给编译器。它有两种形式:

    #[cfg(foo)]
    
    #[cfg(bar = "baz")]

他们也有一些帮助:

    #[cfg(any(unix, windows))]
    
    #[cfg(all(unix, target_pointer_width = "32"))]
    
    #[cfg(not(foo))]

这些可以随意嵌套:

    #[cfg(any(not(unix), all(target_os="macos", target_arch = "powerpc")))]

至于如何启用或禁用这些开关，如果你使用 Cargo，可以在 Cargo.toml 的 [[features] 部分](http://doc.crates.io/manifest.html#the-%5Bfeatures%5D-section) 加以设置：

    [features]
    # no features by default
    default = []
    
    # The “secure-password” feature depends on the bcrypt package.
    secure-password = ["bcrypt"]

当你这样做时,Cargo 传递一个标识给 **rustc**:

    --cfg feature="${feature_name}"

这些 **cfg** 标识的总和将决定哪些得到激活，从而致使哪些代码被编译。让我们看看这段代码：

    #[cfg(feature = "foo")]
    mod foo {
    }

如果我们使用 **cargo build --features "foo"** 编译代码，它将发送 **--cfg feature="foo"** 标识给 **rustc**，且输出中包含 **mod foo**。如果我们定期地使用 **cargo build** 编译它，也不传递额外的标识，就不会存在任何 **foo** 模块。

## cfg_attr

你也可以使用 **cfg_attr** 设置另一个基于 **cfg** 变量的属性：

    #[cfg_attr(a, b)]

如果 **a** 使用 **cfg** 属性设定，和使用 **#[b]** 是相同的。

## cfg!

cfg! [语法扩展](https://doc.rust-lang.org/stable/book/compiler-plugins.html)允许你在你代码中的任何位置使用这些类型标记：

    if cfg!(target_os = "macos") || cfg!(target_os = "ios") {
    println!("Think Different!");
    }

根据配置设置不同，这些在编译时取真或假。