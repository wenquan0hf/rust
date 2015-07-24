# 字符串

对于任何程序员来说，字符串是一个重要的且必须掌握的概念。由于其系统专注的点不同，Rust 的字符串处理系统有点不同于其他计算机语言,。无论何时，当你有一个可变大小的数据结构,事情可能会变得棘手，还有，字符串是一种能重设大小的数据结构。也就是说，Rust 的字符串的工作方式也不同于其他的系统语言，如 C 语言。

让我们深入细节。“字符串”是 Unicode 标量值的序列编码为 utf - 8 字节的流。所有字符串都必须保证为有效的 utf-8 编码序列。此外，不像一些系统语言，这里的字符串是非空终止字符串，并且，可以包含空字节。

Rust 有两个主要类型的字符串： str 和字符串。首先让我们来谈谈 str。它一般被称为“串片”。字符串一般都是&'static str 类型的：

    let string = "Hello there."; // string: &'static str

这个字符串是静态分配的，这意味着它保存在我们的编译程序里面，并且存在于程序运行的整个时间。字符串通过绑定来引用这个静态分配的字符串。字符串片有固定大小,不能突变。

一个字符串，另一方面来说，是一个基于堆的字符串。这个字符串是可生长的,并且保证为 utf-8。通常使用 to_string 方法将一个字符串转换为另一个字符串。

    let mut s = "Hello".to_string(); // mut s: String
    println!("{}", s);
    
    s.push_str(", world.");
    println!("{}", s);
    通过&将字符串放入&str ：
    fn takes_slice(slice: &str) {
    println!("Got: {}", slice);
    }
    
    fn main() {
    let s = "Hello".to_string();
    takes_slice(&s);
    }

将一个字符串转换为 &str 很容易，但将 &str 转换为一个字符串包含分配内存的操作。除非有必要，否则不要这样做！

## 索引

因为字符串都是有效的 UTF-8，字符串不支持索引

    let s = "hello";
    
    println!("The first letter of s is {}", s[0]); // ERROR!!!


 
通常。用 [] 访问向量是非常方便的。但是,因为 utf-8编码的字符串里面的每个字符可以多个字节，你必须检索字符串才可以找到字符串的第 n 个字母。这是一个太过复杂的操作，我们不想被误导。此外，“字母”不是 Unicode 中定义的东西，没错。我们可以把字符串看做一个单独的字节，或 codepoints：

    let hachiko = "忠犬ハチ公";
    
    for b in hachiko.as_bytes() {
    print!("{}, ", b);
    }
    
    println!("");
    
    for c in hachiko.chars() {
    print!("{}, ", c);
    }
    
    println!("");
     
这将输出：

    229,191,160,229,191,172,227,131,143,227,131,129,229,133,172,
    
    忠犬,ハ,チ、公
    
如您所见，这的字节比字符更多。


你可以得到类似于这样的一个索引：
    
    let dog = hachiko.chars().nth(1); // kinda like hachiko[1]

这强调，我们必须跨越整个字符列表。

## 连接

如果你有一个字符串，可以在它的结尾连接一个 str 作为结束：

    let hello = "Hello ".to_string();
    let world = "world!";
    
    let hello_world = hello + world;
    
但是如果你有两个字符串，您需要一个 &：

    let hello = "Hello ".to_string();
    let world = "world!".to_string();
    
    let hello_world = hello + &world;

这是因为 &String 可以自动连接一个 str。这个特点也叫作叫做 ‘Deref coercions’。
