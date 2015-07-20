###基本类型###

Rust 的语言有很多被认为是基本的数据类型。这意味着他们是语言内置的。Rust 是结构化的语言，并且标准库在这些类型至上提供了一些有用的其他的类型，但是这些是最基础的类型。

###Booleans###

Rust 拥有内置的 boolean 类型，名称为 bool。这种类型的变量能够被赋值为 true 或者 false：

```
let x = true;

let y: bool = false;
```

booleans 比较类型常用的方式是在 if 条件中。

在[标准库文档说明](https://doc.rust-lang.org/stable/std/primitive.bool.html)中查看更多的关于 bool 的说明信息。

###char###

char 类型代表的是一个 Unicode 标量值。你可以使用单引号来创建 char 类型变量：

```
let x = 'x';
let two_hearts = '💕';
```

不像其他语言，这意味着在 Rust 中，char 类型不是单个字节而是由四个字节表示。

同样，你可以在[标准库文档](https://doc.rust-lang.org/stable/std/primitive.char.html)中查看更多关于 char 的说明。

###数值类型###

Rust 中有各种数值类型，可以分为这几类：有符号数和无符号数、固定长度和可变长度、浮点数和整数。

这些类型包括两个部分：类别和大小。例如，u16是一个 16 位的无符号类型。更多的比特位能够表示更大的数字。

如果一个数字不能从它字面值推断出他的类型，那么就会默认如下：


```
let x = 42; // x has type i32
let y = 1.0; // y has type f64
```

下面列出了不同的数值类型，同时连接到标准库中的文档说明:

- [i8](https://doc.rust-lang.org/stable/std/primitive.i8.html)
- [i16](https://doc.rust-lang.org/stable/std/primitive.i16.html)
- [i32](https://doc.rust-lang.org/stable/std/primitive.i32.html)
- [i64](https://doc.rust-lang.org/stable/std/primitive.i64.html)
- [u8](https://doc.rust-lang.org/stable/std/primitive.u8.html)
- [u16](https://doc.rust-lang.org/stable/std/primitive.u16.html)
- [u32](https://doc.rust-lang.org/stable/std/primitive.u32.html)
- [u64](https://doc.rust-lang.org/stable/std/primitive.u64.html)
- [isize](https://doc.rust-lang.org/stable/std/primitive.isize.html)
- [usize](https://doc.rust-lang.org/stable/std/primitive.usize.html)
- [f32](https://doc.rust-lang.org/stable/std/primitive.f32.html)
- [f64](https://doc.rust-lang.org/stable/std/primitive.f64.html)

接下来让我们去看看他们的类别：

###有符号和无符号数###

整数类型有两种：有符号和无符号。为了理解他们的差异，首先让我们看一个四位长度的数字。对于有符号数，四位数字能够表示的数据范围是 -8 ～ +7。有符号数用二进制补码表示。对于无符号的四位数字，因为它不需要存储负值，所以可以存储从 0 到 +15。


无符号类型使用 u 表示数据的类别，而有符号数使用 i 表示。i 指的是“整数”。所以 u8 表示的是无符号的 8 位整数，i8 表示的是有符号的 8 位整数。

###固定大小类型###

固定大小的类型由特定数量的比特位数表示。有效位数是 8、16、32、64 位。u32 表示的是无符号 32 位整型数，而 i64 表示的是有符号的 64 位数。

###可变大小类型###

Rust 还提供了大小取决于底层机器指针的大小的类型。这些类型根据大小分为不同的类别，同样有有符号和无符号的类型。比如这两种类型：isize  和 usize。

###浮点类型###

Rust 也有两种浮点类型：f32 和 f64。这些符合于 IEEE-754 单、双精度浮点数的标准。

###数组###

像许多编程语言一样，Rust 有许多类型能够表示顺序对象。最基本的是数组，相同类型的元素和固定大小的顺序对象。默认情况下，数组元素是不可变的。

```
let a = [1, 2, 3]; // a: [i32; 3]
let mut m = [1, 2, 3]; // m: [i32; 3]
```

数组有 [T；N] 类型。我们将在[泛型部分](https://doc.rust-lang.org/stable/book/generics.html)谈论这个 T 符号。N 是一个编译时常量，表示数组的长度。

有一个快捷的方式用相同的值初始化数组的每个元素。在这个例子中，每个元素将被初始化为0：

```
let a = [0; 20]; // a: [i32; 20]
```

你可以利用 a.len() 得到数组中元素的个数：

```
let a = [1, 2, 3];

println!("a has {} elements", a.len());
```

你可以使用数组下标访问数组中某个特定的元素：

```
let names = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]

println!("The second name is: {}", names[1]);
```

下标从 0 开始，就像在大多数编程语言中，第一个元素是 names[0]，第二个元素是 names[1]。上面的示例输出 the second name is: Brian。如果您尝试使用一个不在数组范围内的下标，你会得到一个错误：在运行期访问数组下标越界。这样的访问错误是其他许多编程语言程序中 bug 的来源。

你可以在[标准库文档](https://doc.rust-lang.org/stable/std/primitive.array.html)中找到更多关于数组的说明。

###切片###

“切片”指的是对另一个数据结构的索引(或“视图”)。他们是用于允许安全，高效的访问数组的一部分而不需复制数组的内容。例如，您可能只是想索引文件中某一行并将其读入内存中。从本质上说，切片不是直接创建的，而是来自于现有的变量。切片拥有长度，是可变的也可以设置不可变，并且在许多方面像数组是相似的：

```
let a = [0, 1, 2, 3, 4];
let middle = &a[1..4]; // A slice of a: just the elements 1, 2, and 3
let complete = &a[..]; // A slice containing all of the elements in a
```
切片有 &[T] 类型。我们将会在[泛型部分](https://doc.rust-lang.org/stable/book/generics.html)讲解 T。

你可以在[标准库文档](https://doc.rust-lang.org/stable/std/primitive.slice.html)中找到更多关于切片的内容。

###str###

Rust 的 str 类型是最基本的字符串类型。作为一种无固定大小类型，它本身不是很有用，但是当将其使用在引用符号之后它会会变得作用更大，类似于 &str。这里我们不会进一步讲解这个。

你可以在[标准库文档](https://doc.rust-lang.org/stable/std/primitive.str.html)中找到更多关于 str 的说明。

###元组###

元组是有固定大小的有序列表。类似于：

```
let x = (1, "hello");
```

括号和逗号构成了长度为 2 的元组。如下是相同的代码，但是有类型说明：

```
let x: (i32, &str) = (1, "hello");
```

正如你所看到的，元组的类型看上去和元组一样，但每个位置都有一个类型名称而不是元素值。细心的读者会注意到，元组是异构：在该元组中有一个 i32 类型和一个 &str 类型的元素。在系统编程语言中，字符串比其他语言稍微更复杂点。当下，我们把 &str 当作字符串切片，之后我们会了解更多的。

如果两个元组有相同的元素类型和相同的参数数量，那么你可以将一个元组赋值给另外一个。,当元组有相同的长度时，他们具有相同的参数数量。

```
let mut x = (1, 2); // x: (i32, i32)
let y = (2, 3); // y: (i32, i32)

x = y;
```

你可以使用解析符来访问元组中的字段。如下：

```
let (x, y, z) = (1, 2, 3);

println!("x is {}", x);
```

记得以前我们曾说过左边操作符 let 语句比赋值绑定有强的能力吗？现在我们就可以解释了，我们可以在左边的 let 之后编写一个 pattern，如果它和右边的语句相匹配，那么我们就可以同时进行多个元素的赋值绑定。在这种情况下，let “析构”或者 “分解” 元组，接着进行三个元素的赋值。

这种模式是非常强大的，我们将会在后面的章节再次看到。

你可以在括号中使用逗号来消除是单个元素还是不是单元素的二义性。

```
(0,); // single-element tuple
(0); // zero in parentheses
```

###元组索引###

你还可以使用索引的方式访问元组的字段：

```
let tuple = (1, 2, 3);

let x = tuple.0;
let y = tuple.1;
let z = tuple.2;

println!("x is {}", x);
```

类似于数组索引，下标从零开始，但与数组索引不同的是它使用一个 . 而不是 [] 进行访问。

你可以在[标准库文档](https://doc.rust-lang.org/stable/std/primitive.tuple.html)中找到更多的关于元组的说明。

###函数###

函数同样也是一个类型！如下所示：

```
fn foo(x: i32) -> i32 { x }

let x: fn(i32) -> i32 = foo;
```

在这种情况下，x 是带一个 i32 类型的参数和返回值为 i32 类型的函数指针。
