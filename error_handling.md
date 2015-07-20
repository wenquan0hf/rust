##错误处理##


>不管是人是鼠，即使最如意的安排设计，结局也往往会出其不意。
《致老鼠》 罗伯特·彭斯

有时候,事情会出乎意料的发生错误。重要的是要提前想好应对错误的方法。Rust 有丰富的支持错误处理方法来应对可能(老实说:将会)发生在您的程序中的错误。

主要有两种类型的错误可能发生在你的程序中:故障和异常。让我们谈谈两者之间的区别，然后讨论如何处理它们。接着，将讨论如何将故障升级为异常。

###故障 VS 异常###

Rust 使用两个术语来区分两种形式的错误:故障和异常。故障是可以用某种方式中恢复的错误。异常是一种不能恢复的错误。

我们说的 ”恢复“ 是什么意思？嗯，在大多数情况下，指的是预计一个错误的可能性。例如，考虑 parse 函数:

```
"5".parse();
```

这个方法将一个字符串转换成另一种类型。但因为它是一个字符串,你不能确保转换工作正常执行。例如，执行如下的转换会得到什么?

```
"hello5world".parse();
```

这是行不通的。所以我们知道，这个函数只会对一些特定的输入才能正常工作。这是预期行为。我们称这种错误为故障。

另一方面，有时，有意想不到的错误，或者我们不能恢复它。一个典型的例子是一个断言：

```
assert!(x == 5);
```

我们使用 assert! 说明参数是正确的。如果这不是正确的，那么这个断言就是错误的。错误的话，我们不就能继续在当前状态往下执行了。另一个例子是使用 unreachable!() 宏:

```
enum Event {
	NewRelease;
}

fn probability(_: &Event) -> f64 {
    // real implementation would be more complex, of course
    0.95
}

fn descriptive_probability(event: Event) -> &'static str {
    match probability(&event) {
        1.00 => "certain",
        0.00 => "impossible",
        0.00 ... 0.25 => "very unlikely",
        0.25 ... 0.50 => "unlikely",
        0.50 ... 0.75 => "likely",
        0.75 ... 1.00 => "very likely",
    }
}

fn main() {
    std::io::println(descriptive_probability(NewRelease));
}
```

它将会输出如下的错误：

```
error: non-exhaustive patterns: `_` not covered [E0004]
```

尽管我们已经涵盖所有我们知道的可能情况情况，但是 Rust 不清楚。Rust 不知道概率是 0.0 和 1.0 之间。所以我们添加另一个例子:

```
use Event::NewRelease;

enum Event {
    NewRelease,
}

fn probability(_: &Event) -> f64 {
    // real implementation would be more complex, of course
    0.95
}

fn descriptive_probability(event: Event) -> &'static str {
    match probability(&event) {
        1.00 => "certain",
        0.00 => "impossible",
        0.00 ... 0.25 => "very unlikely",
        0.25 ... 0.50 => "unlikely",
        0.50 ... 0.75 => "likely",
        0.75 ... 1.00 => "very likely",
        _ => unreachable!()
    }
}

fn main() {
    println!("{}", descriptive_probability(NewRelease));
}
```

我们不应该得到 _ 情况，所以我们使用 unreachable!() 宏来说明这个。unreachable!() 比结果给出了不同于 Result 类型的错误。Rust 称这些类型的错误为异常。

###利用 Option 和 Result 处理错误###

表明一个函数可能会失败的最简单方法是使用 option< T >类型。例如，字符串 find 方法试图找到字符串的一个模式串，并返回 Option:

```
let s = "foo";

assert_eq!(s.find('f'), Some(0));
assert_eq!(s.find('z'), None);
```

对这些简单的情况下是可以的，但是在故障的情况下并不会给我们提供很多的信息。如果我们想知道为什么函数发生了故障，怎么办?为此，我们可以使用 Result<T， E>类型。它看起来像这样:

```
enum Result<T,E> {
	ok(T),
    Err(E)
}
```

这个枚举类型由 Rust 本身提供，所以你不需要在你的代码中定义就可以使用它。Ok(T) 变量代表着成功执行，Err(E) 变量代表着执行失败。推荐在大多数情况下返回一个 Result而不是一个 Option 变量。

如下是一个使用 Result 的例子：

```
#[derive(Debug)]
enum Version { Version1, Version2 }

#[derive(Debug)]
enum ParseError { InvalidHeaderLength, InvalidVersion }

fn parse_version(header: &[u8]) -> Result<Version, ParseError> {
    if header.len() < 1 {
        return Err(ParseError::InvalidHeaderLength);
    }
    match header[0] {
        1 => Ok(Version::Version1),
        2 => Ok(Version::Version2),
        _ => Err(ParseError::InvalidVersion)
    }
}

let version = parse_version(&[1, 2, 3, 4]);
match version {
    Ok(v) => {
        println!("working with version: {:?}", v);
    }
    Err(e) => {
        println!("error parsing header: {:?}", e);
    }
}
```

这个函数使用枚举类型变量 ParseError 列举各种可能发生的错误。

[调试](https://doc.rust-lang.org/stable/std/fmt/trait.Debug.html)特点就是让我们使用 {:?} 格式来打印该枚举变量的值。

###遇到 panic！类型的不可恢复错误###

遇到为意料的和不可恢复的的错误时，宏 panic！ 会引起异常。这将崩溃当前线程，并给出一个错误:

```
panic!("boom");
```

当你运行时会输出：

```
thred '<main>' panicked at 'boom', hello.rs:2
```

因为这些类型的情况相对较少见，很少使用恐慌。

###升级故障为异常###

在某些情况下，即使一个函数可能发生故障，我们仍想要把它当作一个异常对待。例如，io::stdin().read_line(&mut buff) 函数在读取某一行时出现错误会返回 Result< usize > 变量。这使我们能够处理它并可能从错误中恢复。

如果我们不想处理这个错误，而宁愿只是中止程序，那么我们可以使用 unwrap() 方法：

```
io::stdin().read_line(&mut buffer).unwrap();
```

如果 Result 变量值是 Err，unwrap() 方法将会产生调用 panic！，输出异常。这基本上是说“给我变量的值，如果出现错误，就让程序崩溃。“这相对于匹配错误并试图恢复的方式可靠性较低，但也大大缩短执行时间。有时,只是崩溃程序是合理的。

有另一种方式比 unwrap() 方法好一点:

```
let mut buffer = String::new();
let input = io::stdin().read_line(&mut buffer)
                       .ok()
                       .expect("Failed to read line");
```
ok() 方法将 Result 转换成一个 Option，并 expect() 和 unwrap() 方法做的事是一样的，不同点在于它需要一个参数，用来输出提示信息。这个消息被传递到底层的 panic！，如果代码出现错误,它提供一个较好的错误消息展示方式。

###使用 try！###

当编写的代码调用很多的返回 Result 类型的函数时，错误处理就变得比较冗长。try! 宏利用堆栈对产生的错误进行引用从而隐藏具体的细节。

将如下的代码：

```
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Info {
    name: String,
    age: i32,
    rating: i32,
}

fn write_info(info: &Info) -> io::Result<()> {
    let mut file = File::create("my_best_friends.txt").unwrap();

    if let Err(e) = writeln!(&mut file, "name: {}", info.name) {
        return Err(e)
    }
    if let Err(e) = writeln!(&mut file, "age: {}", info.age) {
        return Err(e)
    }
    if let Err(e) = writeln!(&mut file, "rating: {}", info.rating) {
        return Err(e)
    }

    return Ok(());
}
```

替换成：

```
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Info {
    name: String,
    age: i32,
    rating: i32,
}

fn write_info(info: &Info) -> io::Result<()> {
    let mut file = try!(File::create("my_best_friends.txt"));

    try!(writeln!(&mut file, "name: {}", info.name));
    try!(writeln!(&mut file, "age: {}", info.age));
    try!(writeln!(&mut file, "rating: {}", info.rating));

    return Ok(());
}
```

用 try！ 封装一个表达式,在成功执行时给 Result 赋值为 Ok，否则赋值为 Err，在这种情况下，在函数未执行完成之前就会返回 Err 值。

值得注意的是，你只能在返回 Result 的函数中使用 try！，这意味着您不能在 main() 中使用 try！，因为 main() 不返回任何值。

try! 利用 [From<Error>](https://doc.rust-lang.org/stable/std/convert/trait.From.html) 来确定在错误的情况下返回的值。
