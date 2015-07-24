# 类型转换

Rust 以其安全性为重点，为不同的类型之间的转换提供了不同的方法。首先，as  用于数据类型安全转换 。相反，transmute 允许类型之间的任意转换，是 Rust 的最危险的特征！

## as

as 关键字可以做基本的转换：

    let x: i32 = 5;
    
    let y = x as i64;

然而，它只允许某些类型的转换：

    let a = [0u8, 0u8, 0u8, 0u8];
    
    let b = a as u32; // four eights makes 32

这个错误如下：

    error: non-scalar cast: `[u8; 4]` as `u32`
    let b = a as u32; // four eights makes 32
    ^~~~~~~~

这是一个“非标量转换” ，因为我们这里有多个值：数组的四个元素。这些类型的转换是非常危险的，因为他们对多个底层结构的实现方式做了假设。为此，我们需要一些更危险的东西。

## transmute

transmute 函数是由内在编译器提供的，它做的非常简单，但非常可怕。它告诉 Rust 把一种类型的值当作它是另一种类型的值。它这样做不管类型检查系统，完全信任你。　　　

在我们前面的例子中，我们知道，一个数组的四个元素 u8 正好表示一个 u32，所以我们做这样的转换。使用 transmute 而不是 as，Rust 代码如下：

    use std::mem;
    
    unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];
    
    let b = mem::transmute::<[u8; 4], u32>(a);
    }
    
为了成功编译我们必须在一个 unsafe 块中封装操作。从技术上讲，只有 mem::transmute 调用自己本身的时候需要在在代码块中。但是在把所有相关的一切封装在内的情况下是可以调用自己的，所以你知道在哪里看。在这种情况下，a 的细节也很重要，它们确实在代码块中。你会看到其它风格的代码，有时上下文是太远，将所有代码封装在 unsafe 不是一个好主意。　　　　

虽然 transmute 确实很少检查，它至少能确保类型是相同大小的。看下面这段代码：

    use std::mem;
    
    unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];
    
    let b = mem::transmute::<[u8; 4], u64>(a);
    }

这个错误是：
    
    error: transmute called on types with different sizes: [u8; 4] (32 bits) to u64
    (64 bits)

除此之外，你需要自己学习相关知识！