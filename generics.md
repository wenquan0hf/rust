#泛型
有时候,写一个函数或数据类型时,我们可能希望它可以作为多种类型的工作参数。幸运的是, Rust给我们提供了一种更好地解决办法:泛型。在类型理论中，泛型类型被称为“参数多态性”,这意味着在给定参数(变量)的情况下，他们是具有多种形式的类型或函数 (“聚”是多个“变形”形式)。

不管怎样,类型理论已经足够充分,让我们看看一些通用的代码。Rust 的标准库提供了一个类型, Option<T>,这是通用的:

    enum Option<T> {
    Some(T),
    None,
    }
    

< T > 部分,正如你之前已经见到过的,表明这是一个通用的数据类型。在我们枚举的声明里面,无论我们在哪里看到 T,我们用泛型中相同的数据类型来作为那个数据类型的替代品。这里有一个使用 Option<T> 的例子,与一些额外的类型注解:

    let x: Option<i32> = Some(5);

在类型声明里面,我们用 Option<i32>。请注意它与 Option<T> 看起来非常类似。所以,在这个特殊的选项 T 中，T的值为 i32。在等号的右边,有一个 some(T),T 的值是 5。因为这是一个 i32,所以双方能够匹配。此时 Rust 运行没有错误。如果他们不匹配,我们会得到一个错误:

    let x: Option<f64> = Some(5);
    // error: mismatched types: expected `core::option::Option<f64>`,
    // found `core::option::Option<_>` (expected f64 but found integral variable)

这并不意味着我们不能让 Option<T>s 的值为 f64 !他们必须匹配才可以: 
    let x: Option<i32> = Some(5);
    let y: Option<f64> = Some(5.0f64);

    
这样很好。因为一个定义,可以用于多个用途。

泛型不必只是通用的一种类型。考虑 Rust 的标准库里面的另一种相似的类型, Result<T, E>:

    enum Result<A, Z> {
    Ok(A),
    Err(Z),
    }

这个类型对这两种类型都是通用的:T 和 E,顺便说一下，大写字母可以是任何你喜欢的字母。我们可以定义 Result<T, E >如下:

    enum Result<A, Z> {
    Ok(A),
    Err(Z),
    }

如果我们想要这样。公约说,第一个通用的参数应该是 T,它对应于“类型”，我们使用 E对应于 error。然而,Rust 并不关心这些。

Result<T, E> 类型的作用是用于返回计算的结果,并能够在发生差错的时候返回一个错误。

###通用函数

我们可以用相似的语法编写带有泛型的函数:

    fn takes_anything<T>(x: T) {
    // do something with x
    }


语法分为两个部分:< T >表示“这个函数对类型 T” 是通用,和 x:T 表示“x 的类型为 T .”
多个参数可以有相同的泛型类型:

    fn takes_two_of_the_same_things<T>(x: T, y: T) {
    // ...
    }
    
我们可以写一个拥有多种类型的版本:

    fn takes_two_things<T, U>(x: T, y: U) {
    // ...
    }

对于特征边界来说通用函数是最有用的,我们将在特征部分讨论它。

###通用结构

你可以存储一个泛型类型的结构:
    
    struct Point<T> {
    x: T,
    y: T,
    }
    
    let int_origin = Point { x: 0, y: 0 };
    let float_originstruct Point<T> {
    x: T,
    y: T,
    }
    
    let int_origin = Point { x: 0, y: 0 };
    let float_origin = Point { x: 0.0, y: 0.0 };

类似于函数,< T > 是我们声明泛型参数的地方,然后,我们在类型声明里使用 x:T。
