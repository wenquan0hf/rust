##特征的对象
当代码涉及多态,需要有一种机制来确定实际上是哪些特定版本在运行。这就是所谓的“调度”。调度的主要形式有两种:静态调度和动态调度。Rust 不仅支持静态调度,它还通过“特征对象”机制支持动态调度。
###背景
在本章的其余部分,我们将需要一个特征和一些实现。让我们先从一个简单的开始,Foo。它有一个方法,该方法将返回一个字符串。

    trait Foo {
    fn method(&self) -> String;
    }

我们将在u8和String里面实现这个特征:

    impl Foo for u8 {
    fn method(&self) -> String { format!("u8: {}", *self) }
    }

    impl Foo for String {
    fn method(&self) -> String { format!("string: {}", *self) }
    }

###静态调度

我们可以利用这一特性的特征边界来执行静态调度:

    fn do_something<T: Foo>(x: T) {
    x.method();
    }
    
    fn main() {
    let x = 5u8;
    let y = "Hello".to_string();
    
    do_something(x);
    do_something(y);
    }

在这里 Rust 使用 “monomorphization” 来执行静态调度。这意味着 Rust 将为 u8 和 String 创建一个特殊版本的 do_something()函数,然后调用这些具体的函数来取代调用 sites 。换句话说,Rust 生成是这样的:

    fn do_something_u8(x: u8) {
    x.method();
    }
    
    fn do_something_string(x: String) {
    x.method();
    }
    
    fn main() {
    let x = 5u8;
    let y = "Hello".to_string();
    
    do_something_u8(x);
    do_something_string(y);
    }
这样会有很大的好处:静态调度允许内联函数调用,因为被调用的函数在编译时是已知的,并且内联是进行良好的优化的关键。静态调度虽然快,但它是折衷之后的结果:代码膨胀,由于许多相同的函数都有二进制副本,并且每种类型的函数都对应一个二进制副本。

另外,编译器并不完美,可能“优化”反而使代码变得更慢。例如,内联函数过于急切地将膨胀指令缓存（缓存规定我们周围的一切）。这是 #[inline] 和 #[inline(always)] 应该被谨慎使用的部分原因,另一个使用动态调度的原因是它有时效率更高。

然而,常见的情况是使用静态调度更高效,并且你总是可以用一个简练的 statically-dispatched 包装器函数进行动态调度,但是用动态调度却不是这样,这意味着静态调用更灵活。可能因为这个原因，标准库更多的使用静态调度。

###动态调度
Rust 通过一个称为特征对象的特性提供动态调度。特征对象,如 &Foo or Box<Foo>,可以是正常值,这个值存储一个任意类型的值，而该类型实现了给定的特征,至于究竟是哪种类型在运行时才能知道。

特征对象可以从指针获得，这个指针指向一个具体的类型,而该类型通过 cast(例如 &x as &Foo)或 coerce(例如,使用 &x 作为函数的参数，函数里面带有 &foo) 实现了特征。

特征对象coercions 和 casts 也服务于指针像 &mut T to &mut Foo 与 Box<T> to Box<Foo>,但都只是在此刻有效。而 Coercions 和 casts 是相同的。


这个操作可以被视为“擦除”编译器对特定类型的指针的认知,因此特征对象有时也被称为类型擦除。


回到上面的例子,我们可以使用相同的特征来执行动态调度与特征的对象的cast:

    fn do_something(x: &Foo) {
    x.method();
    }
    
    fn main() {
    let x = 5u8;
    do_something(&x as &Foo);
    }
    或者通过coerce：:
    fn do_something(x: &Foo) {
    x.method();
    }
    
    fn main() {
    let x = "Hello".to_string();
    do_something(&x);
    }

每种实现 Foo 的类型并不专门指定一个带有个特征对象的函数:只有当一个副本生成时,才会(但不总是如此)导致更少的代码膨胀。然而,这是需要以更慢的虚拟函数调用作为代价的,并且极大地抑制了任何内联的机会和相关优化的发生。

###为选择指针呢?

在默认情况下，Rust 不给指针赋予初值，不像许多其他管理语言那样,所以类型可以有不同的大小。在编译时知道值的大小是很重要的，比如把它作为参数传递给一个函数,又或者把它放进堆栈并且在存储它的堆里面分配（或取消分配）空间给它。

对于 Foo,我们需要有一个值,这个值至少要是一个字符串(24字节)或u8(1个字节),或者是任意其他依赖于箱来实现他们的Foo的类型(任意的字节数)。如果没有存储的值没有指针来指向，就没有办法保证这最后一点行得通，,因为其他那些类型可以是任意大小的。

让指针指向一个值意味着，当我们抛出一个特征对象时，值的大小是无关紧要的,我们只关心指针本身的大小。

###表示

特征的方法可以通过特征对象来调用。特征对象是通过函数指针的一个特殊的记录(由编译器创建和管理)来调用方法的，该记录传统上也被称为“虚表”。

特征对象既简单的又复杂:其核心表示和布局其实很简单,但也有一些复杂的错误消息和令人意外的结果。

让我们以简单的特征对象的运行时间的表示来开始。std::raw 模块包含与内置类型一样复杂的布局结构,包括特征对象:

    pub struct TraitObject {
    pub data: *mut (),
    pub vtable: *mut (),
    }

即特征对象 &Foo 包含数据指针和一个虚表指针。

数据指针指向特征对象所存储的数据(一些未知类型T)，而虚表指针指向对应的虚表(“虚拟方法表”)对应于 T 的 Foo 实现。

vtable 本质上是一个函数指针结构体,指向每个实现方法的具体的机器代码。调用方法trait_object.method() 将检索虚表外部的正确的指针,然后做一个动态调用。例如:

    struct FooVtable {
    destructor: fn(*mut ()),
    size: usize,
    align: usize,
    method: fn(*const ()) -> String,
    }
    
    // u8:
    
    fn call_method_on_u8(x: *const ()) -> String {
    // the compiler guarantees that this function is only called
    // with `x` pointing to a u8
    let byte: &u8 = unsafe { &*(x as *const u8) };
    
    byte.method()
    }
    
    static Foo_for_u8_vtable: FooVtable = FooVtable {
    destructor: /* compiler magic */,
    size: 1,
    align: 1,
    
    // cast to a function pointer
    method: call_method_on_u8 as fn(*const ()) -> String,
    };
    
    
    // String:
    
    fn call_method_on_String(x: *const ()) -> String {
    // the compiler guarantees that this function is only called
    // with `x` pointing to a String
    let string: &String = unsafe { &*(x as *const String) };
    
    string.method()
    }
    
    static Foo_for_String_vtable: FooVtable = FooVtable {
    destructor: /* compiler magic */,
    // values for a 64-bit computer, halve them for 32-bit ones
    size: 24,
    align: 8,
    
    method: call_method_on_String as fn(*const ()) -> String,
    };

每个 vtable 的析构函数的字段都指向一个函数,这个函数将清理 vtable 类型的任何资源，对 u8 来说这几乎是没有什么用处的,但对字符串来说却可以释放内存。如果想要拥有 Box<Foo> 那样的特征对象，这个操作就是必不可少的，当超出范围时，就需要清理 Box 分配以及内部类型。size 和 align 字段存储着被删除类型的大小,及其一致性要求; 由于信息被嵌入到了析构函数，目前这些实际上都未使用, ,但将来会得到应用，因为特征对象逐渐变得更加灵活。

假设我们有一些实现 Foo 的值，然后你会发现 Foo 的显式形式的构建和Foo特征对象的使用看起来有点类似(忽略类型不匹配:无论如何，它们只是指针而已):

    let a: String = "foo".to_string();
    let x: u8 = 1;
    
    // let b: &Foo = &a;
    let b = TraitObject {
    // store the data
    data: &a,
    // store the methods
    vtable: &Foo_for_String_vtable
    };
    
    // let y: &Foo = x;
    let y = TraitObject {
    // store the data
    data: &x,
    // store the methods
    vtable: &Foo_for_u8_vtable
    };
    
    // b.method();
    (b.vtable.method)(b.data);
    
    // y.method();
    (y.vtable.method)(y.data);

如果 b 或 y 拥有特征对象 (Box < Foo >),当他们超出范围时，就会有一个(b.vtable.destructor)(b.data) (分别 y)调用。
