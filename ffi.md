# 外部函数接口 

## 引言 

本指南将使用 [snappy](https://github.com/google/snappy) 压缩/解压库作为引言来介绍编写绑定外部代码。Rust 目前无法直接调用 c++ 库，但是 snappy 包括 C 的接口(记录在 [snappy-c.h](https://github.com/google/snappy/blob/master/snappy-c.h))。

下面是调用外部函数的一个例子，如果你的机器安装了 snappy 它将能够编译通过:

```
extern crate libc;
use libc::size_t;

\#[link(name = "snappy")]
extern {
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
}

fn main() {
    let x = unsafe { snappy_max_compressed_length(100) };
    println!("max compressed length of a 100 byte buffer: {}", x);
}
```

extern 语句块中包含的是外部库中函数签名列表，在这个例子中调用的是平台的 C ABI。#[link(...)] 属性用于指示链接器对 snappy 库进行连接，从而保证库中的符号能够被解析。

外部函数被假定为不安全的，所以当调用他们时，需要利用 unsafe{ } 进行封装，进而告诉编译器被调用的函数中包含的代码是安全的。C 库经常暴露不是线程安全的接口给外部调用，而且几乎任何携带指针参数的函数对于所有的输入都不是有效的，因为这些指可能悬空，并且未经处理的指针可能指向 Rust 内存安全模型之外的区域。

当声明外部函数的参数类型时，Rust 编译器不会检查声明是正确的，所以正确地指定它是在运行时能够正确的绑定的一部分。

extern 块可以扩展到覆盖整个 snappy API:

```
extern crate libc;
use libc::{c_int, size_t};

\#[link(name = "snappy")]
extern {
    fn snappy_compress(input: *const u8,
                       input_length: size_t,
                       compressed: *mut u8,
                       compressed_length: *mut size_t) -> c_int;
    fn snappy_uncompress(compressed: *const u8,
                         compressed_length: size_t,
                         uncompressed: *mut u8,
                         uncompressed_length: *mut size_t) -> c_int;
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
    fn snappy_uncompressed_length(compressed: *const u8,
                                  compressed_length: size_t,
                                  result: *mut size_t) -> c_int;
    fn snappy_validate_compressed_buffer(compressed: *const u8,
                                         compressed_length: size_t) -> c_int;
}
```

## 创建一个安全接口 

原始 C API 需要经过封装之后提供内存安全性，并且才可以使用更高级的概念类似向量。库可以选择只暴露安全、高级接口而隐藏不安全的内部细节。

封装那些使用 slice::raw 模块来访问缓冲区的函数，从而将其当作指针来操作 Rust 的向量。Rust 的向量是内存中的一块连续的区域。向量的长度指的是其中包含元素个数的长度，向量的容量指的是在分配的内存中元素的总大小。长度小于或等于容量。

```
pub fn validate_compressed_buffer(src: &[u8]) -> bool {
    unsafe {
        snappy_validate_compressed_buffer(src.as_ptr(), src.len() as size_t) == 0
    }
}
```

上述 validate_compressed_buffer 封装器使用了不安全的语句块，但它保证对于所有的输入在离开那个 unsafe 函数签名的时候是安全的。

snappy_compress 和 snappy_uncompress 函数更复杂，因为必须要分配一个缓冲区来保存输出数据。

snappy_max_compressed_length 函数可以通过指定最大所需容量用来分配向量空间，接着用该向量来保存输出。向量接着可以被传递到 snappy_compress 函数作为输出参数。输出参数也会被传递，这样通过设置长度之后被压缩的数据的实际长度也可以得到。

```
pub fn compress(src: &[u8]) -> Vec<u8> {
    unsafe {
        let srclen = src.len() as size_t;
        let psrc = src.as_ptr();

        let mut dstlen = snappy_max_compressed_length(srclen);
        let mut dst = Vec::with_capacity(dstlen as usize);
        let pdst = dst.as_mut_ptr();

        snappy_compress(psrc, srclen, pdst, &mut dstlen);
        dst.set_len(dstlen as usize);
        dst
    }
}
```

解压是相似的，因为 snappy 保存未压缩的大小作为压缩格式的一部分，snappy_uncompressed_length 能够返回所需的确切缓冲区大小。

```
pub fn uncompress(src: &[u8]) -> Option<Vec<u8>> {
    unsafe {
        let srclen = src.len() as size_t;
        let psrc = src.as_ptr();

        let mut dstlen: size_t = 0;
        snappy_uncompressed_length(psrc, srclen, &mut dstlen);

        let mut dst = Vec::with_capacity(dstlen as usize);
        let pdst = dst.as_mut_ptr();

        if snappy_uncompress(psrc, srclen, pdst, &mut dstlen) == 0 {
            dst.set_len(dstlen as usize);
            Some(dst)
        } else {
            None // SNAPPY_INVALID_INPUT
        }
    }
}
```

供参考，这里使用的例子也可以在 [GitHub 库](https://github.com/thestinger/rust-snappy) 里面查看。

## 析构函数 

外部库经常更换被调用代码资源的所有权。当这种情况发生时，我们必须使用 Rust 提供的析构函数来提供安全保证的释放这些资源(特别是在恐慌的情况下)。

想要了解更多的析构函数，请查看 [Drop trait](https://doc.rust-lang.org/stable/std/ops/trait.Drop.html)

## Rust 函数调用 C 代码进行回调 

一些外部库需要使用回调函数来报告给调用者他们的当前状态或中间数据。可以通过 Rust 中定义的传递函数与外部库进行通信。当调用 C 代码时，要求回调函数必须使用 extern 标记。

回调函数可以通过注册器发送给调用的 C 库，之后就可以被调用。

一个基本的例子如下：

Rust 代码：

```
extern fn callback(a: i32) {
    println!("I'm called from C with value {0}", a);
}

\#[link(name = "extlib")]
extern {
   fn register_callback(cb: extern fn(i32)) -> i32;
   fn trigger_callback();
}

fn main() {
    unsafe {
        register_callback(callback);
        trigger_callback(); // Triggers the callback
    }
}
```

C 代码：

```
typedef void (*rust_callback)(int32_t);
rust_callback cb;

int32_t register_callback(rust_callback callback) {
    cb = callback;
    return 1;
}

void trigger_callback() {
  cb(7); // Will call callback(7) in Rust
}
```

在这个例子中 Rust 的 main() 函数将调用 C 语言中的 trigger_callback() 函数，接着会在 C 语言中反过来调用 Rust 中的 callback() 函数。

## 针对 Rust 对象的回调 

前面的例子显示了如何在 C 代码中如何回调一个全局函数。然而通常这个回调是针对于 Rust 中某个特定的对象。这个对象可能相应的由 C 对象封装之后的对象。

这个可以通过利用传递一个不安全的指针给 C 库来实现。接着 C 库能够在通知中包含 Rust 对象的指针。此时，允许不安全的访问 Rust 索引对象。

Rust 代码：

```
\#[repr(C)]
struct RustObject {
    a: i32,
    // other members
}

extern "C" fn callback(target: *mut RustObject, a: i32) {
    println!("I'm called from C with value {0}", a);
    unsafe {
        // Update the value in RustObject with the value received from the callback
        (*target).a = a;
    }
}

\#[link(name = "extlib")]
extern {
   fn register_callback(target: *mut RustObject,
                        cb: extern fn(*mut RustObject, i32)) -> i32;
   fn trigger_callback();
}

fn main() {
    // Create the object that will be referenced in the callback
    let mut rust_object = Box::new(RustObject { a: 5 });

    unsafe {
        register_callback(&mut *rust_object, callback);
        trigger_callback();
    }
}
```

C 代码：

```
typedef void (*rust_callback)(void*, int32_t);
void* cb_target;
rust_callback cb;

int32_t register_callback(void* callback_target, rust_callback callback) {
    cb_target = callback_target;
    cb = callback;
    return 1;
}

void trigger_callback() {
  cb(cb_target, 7); // Will call callback(&rustObject, 7) in Rust
}
```

## 异步回调 

在前面例子中给出的是直接调用外部 C 库中提供的函数进行回调。当前线程的控制会从 Rust 转向 C 接着转向 Rust，接着执行回调，最后，触发回调的被调用的函数会在同一线程中执行。

当外部库生成自己的线程，并调用回调时情况就变得更加的复杂。在这些情况下，在回调函数内使用  Rust 中的数据结构是特别不安全的，而且必须使用适当的同步机制。除了经典的同步机制，例如互斥，Rust 中提供了一种可行的方式是使用管道(std::comm)，它会将数据从调用回调的 C 线程中转发到 Rust 中的线程。

如果异步回调的目标是 Rust 地址空间中的一个特殊对象，那么在对象的 Rust 对象被销毁之前在 C 库中肯定不会有更多回调会执行。这个可以通过在对象的析构函数中解除回调关系，并且设计该库确保在正确执行完解除注册之前不会有回调执行。


###链接###

extern 块中的 link 属性提供给 rustc 基本构建块，告诉它如何链接到本地库。有两种可接受的 link 编写形式：

- #[link(name = "foo")]
- #[link(name = "foo", kind = "bar")]

在这两种情况下，foo 是它要连接到本地库的名称，而在第二种情况中 bar 是编译期连接到本地库的类型。目前有三个已知的本地库类型：

- 动态 - #[link(name = "readline")]
- 静态 - #[link(name = "my_build_dependency", kind = "static")]
- 框架 - #[link(name = "CoreFoundation", kind = "framework")]

注意，框架类型仅仅对 OSX 目标平台可用。

不同的 kind 值是为了区分本地库如何不同的进行连接。从连接的角度来看，Rust 编译器创建两种构件：部分(rlib/staticlib)和最终(dylib/binary)。本地动态库和框架属于最终构件范围，而静态库不属于。

如下是几个例子关于如何使用这个模型的：

- 本地构建依赖。有时候在编写 Rust 代码是需要使用一些 C/C++ 代码，但是分布的 C / C++ 代码库格式只是一个负担。在这种情况下,代码将被归档到libfoo。然后锈箱将声明一个依赖通过 `#[link(name = "foo", kind = "static")]`。

不管输出箱的味道,本机静态库将被包含在输出中,这意味着分配本机静态库是不必要的。

- 一个正常的动态依赖关系。常见的系统库(比如readline)都可以在大量的系统,而且经常无法找到这些库的静态副本。当这种依赖是包含在铁锈箱,部分目标(如rlibs)不会链接到库,但当rlib包含在最终的目标(如二进制),本地库将联系在一起。

在 OSX,框架的行为作为一个动态库相同的语义。

## 不安去的语句块 

一些操作，比如引用不安全指针或调用已经被标明为不安全的函数时只允许在不安全的区域内进行。不安全的区域隔离危险，并且向编译期保证不会溢出不安全区域。

不安全的函数，另一方面，必须要显式的表明出。不安全的函数如下所示：

```
unsafe fn kaboom(ptr: *const i32) -> i32 { *ptr }
```

这个函数只能从一个不安全的区域中调用或被其他不安全的函数调用。

## 访问外部全局变量 

外部 API 经常导出全局变量，这样可以做一些类似于跟踪全局状态的事情。为了访问这些变量，你在 extern 语句块中声明他们时要使用关键字 static：

```
extern crate libc;

\#[link(name = "readline")]
extern {
    static rl_readline_version: libc::c_int;
}

fn main() {
    println!("You have readline version {} installed.",
             rl_readline_version as i32);
}
```

或者，您可能需要使用外部接口来改变全局状态。为了做到这一点，在声明他们时使用 mut，这样就可以修改他们了。

```
extern crate libc;

use std::ffi::CString;
use std::ptr;

\#[link(name = "readline")]
extern {
    static mut rl_prompt: *const libc::c_char;
}

fn main() {
    let prompt = CString::new("[my-awesome-shell] $").unwrap();
    unsafe {
        rl_prompt = prompt.as_ptr();

        println!("{:?}", rl_prompt);

        rl_prompt = ptr::null();
    }
}
```

注意，所有与 static mut 类型的变量交互是不安全的，包括读和写。为了处理全局可变状态你需要多花点心思。

## 外部调用约定 

大多数外部代码暴露了 C ABI，并且 Rust 默认情况下调用外部函数时使用的是 C 平台调用约束。一些外部函数，尤其是 Windows API，使用的是其他调用约定。Rust 提供了一种方法来告诉编译器它使用的是哪个约定：

```
extern crate libc;

\#[cfg(all(target_os = "win32", target_arch = "x86"))]
\#[link(name = "kernel32")]
\#[allow(non_snake_case)]
extern "stdcall" {
    fn SetEnvironmentVariableA(n: *const u8, v: *const u8) -> libc::c_int;
}
```

下面的适用于整个 extern 块。Rust 中支持的ABI 约束列表如下：

- stdcall
- aapcs
- cdecl
- fastcall
- Rust
- rust-intrinsic
- system
- C
- win64

上面列表中的大部分 abis 是不需要解释的，但 system 这个 abi 可能看起来有点奇怪。这个约束的意思是选择与任何与目标库合适的 ABI 进行交互。例如，在 win32 x86 体系结构中，这意味着 abi 将会选择 stdcall。然而在 `x86_64` 中，windows 使用 C 调用协定，因此将会使用 C 的标准。也就是说，在前面的例子中，我们可以在 extern 中使用 “system”{...} 来定义 所有 windows 系统中的块，而不仅仅是 x86 的。

## 与外部代码的交互 

只要 `#[repr(C)]` 这个属性应用在代码中，Rust 保证的 struct 的结构与平台的表示形式是兼容的。`#[repr(C、包装)]` 可以用来布局 sturct 的成员没而不需要有填充元素。`#[repr(C)]` 也适用于枚举类型。

Rust 中的 `boxes(Box<T>)` 使用非空指针作为句柄指向其中所包含的对象。然而，他们不应该手动创建的，因为它们是由内部分配器管理。引用可以安全地假定指针非空指向该类型。然而，破坏 borrow 的检查或易改变的改规则不能保证是安全的，所以如果有必要请使用原始指针(*)，因为编译器不能对他们呢进行过多的假设。

向量和字符串共享相同的基本的内存布局，并且可以通过 vec 和 str 模块与 C APIs 进行交流。然而，字符串不是以 `\0` 作为它的结束符。如果你想要使用一个空终结符字符串与 C 语言的交互，此时你应该使用 std::ffi 模块中的 CString 类型。

标准库包括类型别名和函数的定义，对于 C 标准库位于 libc 模块中，而且 Rust 默认情况下已经链接了 libc 和 libm 库。

## 可空指针优化 

某些类型的定义不为空。这包括引用类型`(&T、&mut T)，boxes(Box<T>)`，和函数指针`(extern "abi" fn())`。当与 C 交互时，经常使用的指针可能为空。特殊的情况下，泛型枚举中仅仅包含两个亮亮，其中一个不包含数据，另一个包含单个字段，这个能够进行空指针优化。当这个枚举类型被一个非空类型初始化时，它就表示一个指针，并且那个没有数值的变量就成为空指针。因此，`Option<extern "C" fn(c_int) -> c_int>` 展示了一个表示空函数指针是如何使用 C ABI。

## C 语言中调用 Rust 代码 

你可能想要在 C 中调用 Rust 代码，并且编译。这也好似相当容易，但是需要几件事：

```
\#[no_mangle]
pub extern fn hello_rust() -> *const u8 {
    "Hello, world!\0".as_ptr()
}
```

extern 让这个函数符合 C 调用函数的约束，就如上面说的“外部函数调用约束”。no_mangle 属性关闭了 Rust 的名称纠正，因此这里是很容易的进行连接的。
