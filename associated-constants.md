# 相关常量 #

使用 `associated_consts` 功能，用户可以采用如下方式来定义常量：   


    #![feature(associated_consts)]
    
    trait Foo {
    const ID: i32;
    }
    
    impl Foo for i32 {
    const ID: i32 = 1;
    }
    
    fn main() {
    assert_eq!(1, i32::ID);
    }


任何 `Foo` 的实现都必须定义 `ID`. 如果没有定义：


    #![feature(associated_consts)]
    
    trait Foo {
    const ID: i32;
    }
    
    impl Foo for i32 {
    }

上述代码出现下面的提示：   



`error: not all trait items implemented, missing: `ID` [E0046]
     impl Foo for i32 {
     }`


默认的值可以采用如下实现：

    #![feature(associated_consts)]
    
    trait Foo {
    const ID: i32 = 1;
    }
    
    impl Foo for i32 {
    }
    
    impl Foo for i64 {
    const ID: i32 = 5;
    }
    
    fn main() {
    assert_eq!(1, i32::ID);
    assert_eq!(5, i64::ID);
    }


正如用户看到那样，当实现 `Foo` 的时候，用户可以不实现 `i32`，它就会使用默认值。至于 `i64`，我们也可以增加我们自己的定义。

相关常量并不是必须与一个特性相关。`struct` 的`impl`块也可以很好的工作，如下：    

    #![feature(associated_consts)]
    
    struct Foo;
    
    impl Foo {
    pub const FOO: u32 = 3;
    }


