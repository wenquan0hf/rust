# Lang 项目  

**注意：**lang 项目一般是由 Rust 发布的封装提供的，lang 项目本身具有不稳定的接口。最好使用官方发布的封装，不要使用自己编写的 lang 项目。

`rustc` 编译器具有可插拔的操作功能，它并不是硬编码成语言的，而是被以库的形式实现，并且以特殊标记来告知编译器它的存在。这个标记就是  `#[lang = "..."]`，并且有多种多样的值，多种多样的 lang 项目。

例如，`Box` 指针需要两个 lang 项目，一个是分配，一个是回收。一个使用 `Box` 的独立程序可以通过 `malloc` 和 `free` 来动态的管理内存分配。    

	#![feature(lang_items, box_syntax, start, no_std, libc)]
	#![no_std]

	extern crate libc;

	extern {
	fn abort() -> !;
	}

	#[lang = "owned_box"]
	pub struct Box<T>(*mut T);

	#[lang = "exchange_malloc"]
	unsafe fn allocate(size: usize, _align: usize) -> *mut u8 {
	let p = libc::malloc(size as libc::size_t) as *mut u8;

	// malloc failed
	if p as usize == 0 {
	abort();
	}

	p
	}
	#[lang = "exchange_free"]
	unsafe fn deallocate(ptr: *mut u8, _size: usize, _align: usize) {
	libc::free(ptr as *mut libc::c_void)
	}

	#[start]
	fn main(argc: isize, argv: *const *const u8) -> isize {
	let x = box 1;

	0
	}

	#[lang = "stack_exhausted"] extern fn stack_exhausted() {}
	#[lang = "eh_personality"] extern fn eh_personality() {}
	#[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }


注意 `abort` 的使用：`exchange_malloc` lang 项目可以被认为是返回一个有效的指针，且需要内部的检查。  

其他的由 lang 项目提供的功能包括：   

- 基于特征的可重载操作符：操作符 ==, <, * 和 + 都被 lang 项目标记起来。这些特殊的操作符对应于 `eq`, `ord`, `deref` 和 `add`.   
   
- 展开堆栈和一般性错误。`eh_personality`, `fail` 和 `fail_bounds_checks` lang项目。    
   
- `std::marker` 中的特点用于表示不同类型，lang 项目 `send`, `sync` 和 `copy`。    

- `std::marker`中的标记类型和变量指示符，lang 项目有 covariant_type, contravariant_lifetime    

lang 项目会被编译器以消极的方式加载。如果用户不使用 Box ，那么就不需要为 `exchange_malloc` 和 `exchange_free`定义函数。当以个lang 项目在当前代码中存在，那么 `rustc` 就会发出错误提示。