# 内敛函数

**注意：**内敛函数永远不具有稳定接口，所以最好直接使用libcore的稳定接口，而不是直接使用内敛函数。

使用 `rust-intrinsic` ABI 可以使得内敛函数类似于外部函数接口方法一样去引入。比如，如果一段程序希望不依赖于上下文，同时又希望能够在不同类型见切换，然后高效的执行指针运算，那么这段代码可以通过如下的声明来引入这些方法。

    extern "rust-intrinsic" {
    fn transmute<T, U>(x: T) -> U;
    
    fn offset<T>(dst: *const T, offset: isize) -> *const T;
    }


调用其他任何外部函数接口方法都是不安全的。
