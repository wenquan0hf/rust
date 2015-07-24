# 内联汇编

由于低水平的操作和性能，人们希望直接控制 CPU。Rust 通过 asm！宏来支持使用内联汇编。语法大致匹配 GCC & Clang ：

    asm!(assembly template
       : output operands
       : input operands
       : clobbers
       : options
       );

任何 asm 的使用是特征封闭的（需要库的 #![feature(asm)] 允许），当然需要一个 unsafe 块。

    注意:这里给出了 x86 和 x86-64 支持的例子，但所有平台都支持

## 汇编模板

assembly template 是唯一所需的参数，它必须是一个文字字符串（例如，""）

    #![feature(asm)]
    
    #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
    fn foo() {
    unsafe {
    asm!("NOP");
    }
    }
    
    // other platforms
    #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
    fn foo() { /* ... */ }
    
    fn main() {
    // ...
    foo();
    // ...
    }

（从现在开始省略 feature(asm) 和 #[cfg]。）

输出操作数，输入操作数，clobber 和选择项都是可选的，但是你必须添加正确的数量的 : 如果你跳过它们：

    asm!("xor %eax, %eax"
    :
    :
    : "{eax}"
       );

空格也没有关系：

    asm!("xor %eax, %eax" ::: "{eax}");

## 操作数

输入和输出操作数遵循相同的格式："constraints1"(expr1), "constraints2"(expr2), ..."。输出操作数表达式必须是可变左值，或者没有分配内存：

    fn add(a: i32, b: i32) -> i32 {
    let c: i32;
    unsafe {
    asm!("add $2, $0"
     : "=r"(c)
     : "0"(a), "r"(b)
     );
    }
    c
    }
    
    fn main() {
    assert_eq!(add(3, 14159), 14162)
    }

如果你想在这个位置上使用真正的操作数，然而，你需要把花括号 { } 放在你想要的的寄存器两边，你需要加具体操作数的大小。对于低水平的编程这是非常有用的，在程序中使用哪个寄存器很重要：

    let result: u8;
    asm!("in %dx, %al" : "={al}"(result) : "{dx}"(port));
    result

## Clobbers　　　　

一些指令修改有可能持有不同值的寄存器，所以我们使用超时列表来指示编译器不承担加载到寄存器将保持有效的任何值。

    // Put the value 0x200 in eax
    asm!("mov $$0x200, %eax" : /* no outputs */ : /* no inputs */ : "{eax}");

输入和输出寄存器不需要被列出来，因为信息已经被给定约束传达。否则，任何其他被隐式或显式地使用的寄存器应该列出。　　　　

如果内联会更改代码寄存器， cc 应该指定为一个 clobber。同样，如果内联会修改内存，memory 还应该被指定。

## 选择项

最后一部分，options 是 Rust 特有的。形式是逗号分隔字符串（例如：:"foo", "bar", "baz"）。这是用于指定内联汇编的一些额外的信息：　　　　

当前有效的选项是：　　　　

1. *volatile* -这类似于在 gcc/clang 中指定\_ \_asm\_\_ \_\_volatile\_\_(...) 。　　
2. *alignstack*-某些指定堆的对齐某种方式（例如，SSE）的指令并说明这个指示编译器插入其通常堆栈对齐的代码的指令。　
3. *intel* -使用 intel 语法而不是默认的 AT&T。

        let result: i32;
        unsafe {
           asm!("mov eax, 2" : "=`{eax}"(result) : : : "intel")
        }  println!("eax is currently {}", result);
    

