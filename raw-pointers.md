# 原始指针 

Rust 在标准库有许多不同的智能指针类型，但是有两种特别的类型。Rust 的安全来自于编译时检查，但原始指针没有这样的保证，使用起来不安全。　　　　

\*const T 和 \*mut T 在 Rust 中被称为“原始指针”。有时，当写库的某些类型时，出于某种原因你需要绕过 Rust 的安全保证。在这种情况下，你可以使用原始指针来实现你的库，同时为给用户一个安全接口。例如，允许 * 指针起别名，允许他们被用来写共享类型，甚至线程安全共享内存类型（ Rc<T>和 Arc<T> 类型在 Rust 中都被完全实现）。

这里要记住原始指针与其他指针类型是不同的地方。他们：

- 不能保证指有效内存，甚至不保证非空（不像 Box 和 & ）；　　
- 不像 Box ，没有任何自动清理功能，所以需要手动管理资源； 　　
- 是原始旧数据，也就是说，他们不转移指向，不像 Box ，因此 Rust 编译器不能防止像 use-after-free 一样的漏洞； 　　
- 与 & 不用，缺乏任何形式的生命周期，所以编译器无法推断悬空指针；　　
- 不能保证别名使用或者易变性，不同于直接通过 *const T 不能产生变化。

## Basics

创建一个原始指针是绝对安全的：

    let x = 5;
    let raw = &x as *const i32;
    
    let mut y = 10;
    let raw_mut = &mut y as *mut i32;

然而，非关联化的指针是不安全的。这是行不通的：

    let x = 5;
    let raw = &x as *const i32;
    
    println!("raw points at {}", *raw);

它会给出这样的错误：

    error: dereference of unsafe pointer requires unsafe function or block [E0133]
     println!("raw points at{}", *raw);
     ^~~~

原始指针废弃时，你承担这样的后果，那就是它不指向那个正确的地方。因此，你需要 unsafe：

    let x = 5;
    let raw = &x as *const i32;
    
    let points_at = unsafe { *raw };
    
    println!("raw points at {}", points_at);

要了解更多关于原指针的操作，请看 API 文档。

## FFI

对 FFI 来说原指针是很有用的：Rust 的 \*const T 和 \*mut T 与 C 的 const T* 和 T* 是相似的。更多关于它的使用，请看 FFI 章节。

## 引用和原始指针

在运行时，一个原始指针 \* 和一个指向同一块数据的引用具有相同的表示。事实上，一个 &T 引用将隐式强制转换为安全代码中的 \*const T 原始指针并且与常量 mut 相似（两种强制转换可以显式地被执行，值分别是 *const T 和  *mut T）。　　　　

相反，从 *const 指向引用 & 是不安全的。 &T 总是有效的，因此至少，原始指针 *const T 必须指向一个 T 类型的有效实例。此外，由此产生的指针必须满足引用的别名使用和可变性规则。编译器假定这些属性对任何引用都是真的，不管他们是如何创建的，因此任何从原始指针的转换都认为它们有这些属性。程序员必须保证这一点。　　　　

推荐的转换方法是

    let i: u32 = 1;
    
    // explicit cast
    let p_imm: *const u32 = &i as *const u32;
    let mut m: u32 = 2;
    
    // implicit coercion
    let p_mut: *mut u32 = &mut m;
    
    unsafe {
    let ref_imm: &u32 = &*p_imm;
    let ref_mut: &mut u32 = &mut *p_mut;
    }

&*x 解除引用要使用 transmute。transmute 是非常强大的，更受限制的操作也能正确使用；例如，它要求 x 是一个指针（而不像 transmute）。