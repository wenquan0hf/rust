##注释##

既然我们对函数有了一定了解之后，那么学习下如何写注释是不错的。注释的作用在于它能够帮助其他的程序员更好的理解你的代码。而编译期通常会忽视他们。

Rust 中有两种你应该学习的注释方式：行注释和文档注释。

```
// Line comments are anything after ‘//’ and extend to the end of the line.

let x = 5; // this is also a line comment.

// If you have a long explanation for something, you can put line comments next
// to each other. Put a space between the // and your comment so that it’s
// more readable.
```

另一种注释是文档注释。文档注释使用 /// 而不是 //，并且在该注释部分中支持 markdown 注解文法：


	/// Adds one to the number given.
	///
	/// # Examples
	///
	/// ```
	/// let five = 5;
	///
	/// assert_eq!(6, add_one(5));
	/// ```
	fn add_one(x: i32) -> i32 {
 	   x + 1
	}

在编写文档注释时提供一些程序使用的例子是非常有用的。你会注意到在这里我们使用了一个新的宏： assert_eq!。它将两个值进行比较，如果两个值不相等就会调用 panic!。这是非常有用的文档说明。还有另一个宏，assert!，如果它的参数值是 false，那么同样会触发 panic！ 的执行。

您可以使用 rustdoc 工具从这些文档注释生成HTML文档，并运行测试代码来尝试一下!
