# 不依赖 stdlib #

默认情况下，`std` 会链接到每一个 Rust 的封装对象。在一些场景下，这并不是很理想的，在需要的情境下使用 `#![no_std]` 属性使得程序可以独立于 `stdlib`。    

    // a minimal library
    #![crate_type="lib"]
    #![feature(no_std)]
    #![no_std]

很显然，这种方式比仅仅使用库有更长的声明周期：一个是使用 `#[no_std]` 来开始程序的执行，控制程序开始指针可以有两种方式：使用 `#[start]` 属性或覆盖 C `main` 函数。    

以 `#[start]` 标记的函数可以以C语言的方式来传递命令行参数:   


    #![feature(lang_items, start, no_std, libc)]
    #![no_std]
    
    // Pull in the system libc library for what crt0.o likely requires
    extern crate libc;
    
    // Entry point for this program
    #[start]
    fn start(_argc: isize, _argv: *const *const u8) -> isize {
    0
    }
    
    // These functions and traits are used by the compiler, but not
    // for a bare-bones hello world. These are normally
    // provided by libstd.
    #[lang = "stack_exhausted"] extern fn stack_exhausted() {}
    #[lang = "eh_personality"] extern fn eh_personality() {}
    #[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }


为了覆盖插入式编辑器的 `main` 函数。一种方法是使用 `#![no_main]` 来关闭，然后使用正确 ABI 和名字来创建合适的标示，这些也需要覆盖编译器的名字。     
    

	`#![feature(no_std)]
	#![no_std]
	#![no_main]
	#![feature(lang_items, start)]

	extern crate libc;

	#[no_mangle] // ensure that this symbol is called `main` in the output
	pub extern fn main(argc: i32, argv: *const *const u8) -> i32 {
    0
	}

	#[lang = "stack_exhausted"] extern fn stack_exhausted() {}
	#[lang = "eh_personality"] extern fn eh_personality() {}
	#[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }`   


编译器目前已经制造了一些假定的符号，它们可以在可执行文件来调用。通常，这些函数是由标准库提供的，如果没有它的话，用户就必须定义自己的。

这三个函数中的第一个，`stack_exhausted`，被调用何时堆栈溢出。此函数关于如何被调用和它必须做什么存在一些约束和限制。但是，如果堆栈限制寄存器没有被维护，然后一个线程经常有个无限的堆栈且这个函数不会被触发。

这三个函数中的第三个函数，`eh_personality`，用于编译器的错误机制。这通常会被映射到 GCC 的个人函数（参照 [libstd](http://doc.rust-lang.org/stable/std/rt/unwind/) 的实现来获取更多的信息），不会触发错误的对象就被认为是该函数没有被调用。最后一个函数，`panic_fmt`，也是用于编译器的错误机制的。   


## 使用 libcore ##

**注意：**核心库的结构是不稳定的，这里特别建议尽量在能使用标准库的情况下就使用标准库。


在上述技术的基础上，我们就可以运行一些简单的 Rust 代码。标准库可以提供一些更理想的函数，但是，必须在 Rust 代码中主动去建立。在某些情况下，标准库也不是很有效，那么，可以使用 [libcore](http://doc.rust-lang.org/stable/core/).


核心库具有较少的依赖性，并且比标准库还精简。此外，核心库为编写惯用且高效的 Rust 代码提供了很多必要的函数。

举个例子，下面是关于 C 中的两个向量进行点乘的程序，这里使用了惯用的 Rust 实践。    

	`#![feature(lang_items, start, no_std, core, libc)]
	#![no_std]

	extern crate core;

	use core::prelude::*;

	use core::mem;

	#[no_mangle]
	pub extern fn dot_product(a: *const u32, a_len: u32,
                          b: *const u32, b_len: u32) -> u32 {
    use core::raw::Slice;

    // Convert the provided arrays into Rust slices.
    // The core::raw module guarantees that the Slice
    // structure has the same memory layout as a &[T]
    // slice.
    //
    // This is an unsafe operation because the compiler
    // cannot tell the pointers are valid.
    let (a_slice, b_slice): (&[u32], &[u32]) = unsafe {
        mem::transmute((
            Slice { data: a, len: a_len as usize },
            Slice { data: b, len: b_len as usize },
        ))
    };

    // Iterate over the slices, collecting the result
    let mut ret = 0;
    for (i, j) in a_slice.iter().zip(b_slice.iter()) {
        ret += (*i) * (*j);
    }
    return ret;
	}

	#[lang = "panic_fmt"]
	extern fn panic_fmt(args: &core::fmt::Arguments,
                    file: &str,
                    line: u32) -> ! {
    loop {}
	}

	#[lang = "stack_exhausted"] extern fn stack_exhausted() {}
	#[lang = "eh_personality"] extern fn eh_personality() {}`


注意，这里有个与上例中不同的附加lang 项目，`panic_fmt`.它必须由 libcore 的使用者来ID来定义，因为，核心库声明了这个警告，但是并没有定义它。这里的 `panic_fmt` lang 项目 就是警告的对象定义。它必须满足应用不会退出的条件。  

正如上例中看到的那样，核心库就是用于在多种独立于平台的应用场景下支持 Rust.未来的库,如liballoc、会在libcore的基础上添加更多的功能，使的可以应用于其他特定的平台,但不变的是仍然会比标准库更精简便携。