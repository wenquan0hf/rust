## 测试

程序测试是一个非常有效的方法，它可以有效的暴漏程序中的缺陷，但对于暴漏缺陷来说，这还是远远不够的。   
—— Edsger W. Dijkstra，"卑微的程序员" (1972)

让我们来谈谈如何测试 Rust 代码。我们将谈论不是什么测试 Rust 代码正确的方法。关于正确和错误地编写测试的方式有很多的流派。所有这些方法都使用相同的基本工具，因此，我们将向您展示使用它们的语法。

### 测试属性

Rust 中一个最简单的测试是一个函数，它使用 test 属性注释。让我们使用 Cargo 做一个叫加法器的新项目：

```
    $ cargo new adder
    $ cd adder
```

当你做一个新项目时，Cargo 将自动生成一个简单的测试。下面即是 src/lib.rs 的内容：

```
    #[test]
    fn it_works() {
    }
```

注意 `#[test]`。该属性表明，这是一个测试函数，目前还没有函数体。我们可以使用 Cargo test 运行这个测试：

```
    $ cargo test
       Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a
    
    running 1 test
    test it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
       Doc-tests adder
    
    running 0 tests
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Cargo 编译和运行我们的测试。这里有两组输出：一个用于我们写的测试，另一个用于文档测试。稍后我们将讨论这一问题。现在，让我们来看看这一行：

```
    test it_works ... ok
```

注意 **it_works**。这是来自我们的函数的名称:

```
    fn it_works() {
```

我们还得到一个总结:

```
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
```

那么为什么我们的测试能够通过呢?任何非 panic 的测试都可以通过，任何 panic 的测试都会失败。让我们来看一个失败的测试：

```
    #[test]
    fn it_works() {
    	assert!(false);
    }
```

**assert!** 一种 Rust 提供的宏，它需要一个参数：如果参数是**true**，什么也不会发生。如果参数是 false，它就成为 **panic！** 的。让我们再次运行我们的测试：

```
    $ cargo test
       Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a
    
    running 1 test
    test it_works ... FAILED
    
    failures:
    
    ---- it_works stdout ----
    thread 'it_works' panicked at 'assertion failed: false', /home/steve/tmp/adder/src/lib.rs:3
    
    
    
    failures:
    it_works
    
    test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured
    
    thread '<main>' panicked at 'Some tests failed', /home/steve/src/rust/src/libtest/lib.rs:247
```

Rust 表明我们的测试失败：

```
    test it_works ... FAILED
```

反映在结论中就是：

```
    test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured
```

还可以得到一个非零的状态代码：

```
    $ echo $?
    101
```

如果你想将 **cargo test** 集成到其他工具，这是非常有用的。

我们可以用另一个属性：should_panic 转化我们的测试的失败：

    #[test]
    #[should_panic]
    fn it_works() {
    	assert!(false);
    }

如果我们 **panic！**，这个测试会成功，如果我们完成，则测试会失败。让我们来试一试：

```
    $ cargo test
       Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a
    
    running 1 test
    test it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
       Doc-tests adder
    
    running 0 tests
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Rust 提供另一个宏 assert_eq!，用来比较两个参数是否相等：

```
    #[test]
    #[should_panic]
    fn it_works() {
    	assert_eq!("Hello", "world");
    }
```

这个测试是否可以通过?因为存在 should_panic 属性，它可以通过：

```
    $ cargo test
       Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a
    
    running 1 test
    test it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
       Doc-tests adder
    
    running 0 tests
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

**should_panic** 测试很脆弱。很难保证测试不会因为一个意想不到的原因而失败。为了解决这个问题，可以在 **should_panic** 属性中添加一个可选的参数：**expected**。测试工具将确保错误消息包含提供的文本。上面示例的安全版本是：

```
    #[test]
    #[should_panic(expected = "assertion failed")]
    fn it_works() {
    	assert_eq!("Hello", "world");
    }
```

这就是所有的基础让我们来编写一个“真正”的测试：

```
    pub fn add_two(a: i32) -> i32 {
    	a + 2
    }

    #[test]
    fn it_works() {
    	assert_eq!(4, add_two(2));
    }
```

这是 assert_eq! 的一个非常常见的用法：使用一些已知的参数调用某些函数并与预期的输出比较。

### 测试模块

有一种方式，以这种方式我们现有的例子都是不符合惯例的：它缺少测试模块。我们的示例的惯用写作方式，如下所示:

```
    pub fn add_two(a: i32) -> i32 {
    a + 2
    }
    
    #[cfg(test)]
    mod tests {
    use super::add_two;
    
    #[test]
    fn it_works() {
    assert_eq!(4, add_two(2));
    }
    }
```

这里有一些变化。第一个是引入带有 **cfg** 属性的 **mod tests**。模块允许我们对所有的测试进行分组，如果需要也可以定义 helper 函数，这个函数不会成为我们 crate 的一部分。如果目前我们试图运行这些代码，**cfg** 属性只会编译我们的测试代码。这可以节省编译时间，也保证了我们构建的测试是完全正常的。

第二个变化是 **use** 声明。因为我们在一个内部模块中，我们需要将我们的测试函数设置范围。如果你有一个大的模块，这可能就会很恼人，所以这是 glob 属性的一种常见的使用方式。让我们改变我们的 `src/lib.rs` 以便能够使用它：

```
    pub fn add_two(a: i32) -> i32 {
    a + 2
    }
    
    #[cfg(test)]
    mod tests {
    use super::*;
    
    #[test]
    fn it_works() {
    assert_eq!(4, add_two(2));
    }
    }
```

注意 use 行的不同使用方式。现在，我们来运行我们的测试：

```
    $ cargo test
    Updating registry `https://github.com/rust-lang/crates.io-index`
       Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a
    
    running 1 test
    test tests::it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
       Doc-tests adder
    
    running 0 tests
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

看，运转起来了!

当前的惯例是使用测试模块 “unit-style” 测试。任何只测试一个小功能都是有意义的。如果用 “integration-style” 测试替代会怎么样呢?为此，我们引出了测试目录。

### 测试目录

为了编写集成测试，让我们做一个测试目录，并把一个 `tests/lib.rs` 文件放在里面，这是它的内容：

```
    extern crate adder;
    
    #[test]
    fn it_works() {
    assert_eq!(4, adder::add_two(2));
    }
```

这类似于我们之前的测试，但略有不同。在代码顶部有一个 **extern crate adder**。这是因为在测试目录里测试是一个完全独立的箱，所以我们需要导入我们的函数库。这也是为什么 tests 是一个编写集成风格测试的合适的地方：他们使用函数库和其他消费者。

让我们运行它们:

```
    $ cargo test
       Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a
    
    running 1 test
    test tests::it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
     Running target/lib-c18e7d3494509e74
    
    running 1 test
    test it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
       Doc-tests adder
    
    running 0 tests
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

现在我们有三个部分：我们之前的测试在运行，现在这个新的也在运行。

这就是所有的 **tests** 目录。这里不需要测试模块，因为整件事都是专注于测试的。

让我们最后检查一下第三部分：文档测试。

### 文档测试

没有什么是比带有示例的文档更好的了。没有什么是比不能真正工作的例子更糟的了，一直以来文档编写已经改变了代码习惯。为此，Rust 支持自动运行你的文档中的示例。这里有一个完整的 `src/lib.rs` 的例子：

<pre><code>
//! The `adder` crate provides functions that add numbers to other numbers.
//!
//! # Examples
//!
//! ```
//! assert_eq!(4, adder::add_two(2));
//! ```

/// This function adds two to its argument.
///
/// # Examples
///
/// ```
/// use adder::add_two;
///
/// assert_eq!(4, add_two(2));
/// ```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
</code></pre>

注意：模块级文档使用 `//!`，函数文档使用 `///`。Rust 的文档支持 Markdown 中评论，所以三重斜线标志代码块。包含 # Examples 部分是一种惯例，以下所示。

让我们再次运行测试:

```
    $ cargo test
       Compiling adder v0.0.1 (file:///home/steve/tmp/adder)
     Running target/adder-91b3e234d4ed382a
    
    running 1 test
    test tests::it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
     Running target/lib-c18e7d3494509e74
    
    running 1 test
    test it_works ... ok
    
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
    
       Doc-tests adder
    
    running 2 tests
    test add_two_0 ... ok
    test _0 ... ok
    
    test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured
```

现在我们运行了所有三种测试！注意这些测试文档的名称：the_0 生成模块测试，add_two_0 生成功能测试。当你添加更多的例子，这些名字会自动增量（例如 add_two_1）。