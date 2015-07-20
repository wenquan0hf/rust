##If let 
If let 允许你把 if 和 let 结合到一起,来减少某些类型的模式匹配所需的开销。

例如,有某种 Option<T>。如果它是 Some<T>，我们希望在它上面调用一个函数，如果不是，则什么也不做。就像下面这样：

    match option {
    Some(x) => { foo(x) },
    None => {},
    }


在这里我们不一定非要使用匹配,例如,我们可以使用 if

    if option.is_some() {
    let x = option.unwrap();
    foo(x);
    }

这些选项都不是特别有吸引力。我们可以用 if let 语句以更好的方式做同样的事情:

    if let Some(x) = option {
    foo(x);
    }

如果一个模式匹配成功,它将给模式的标识符绑定任意合适的值,然后评估表达式。如果模式不匹配,则什么也不去做。

当模式不匹配时，如果你希望去做别的事情,您可以使用else:

    if let Some(x) = option {
    foo(x);
    } else {
    bar();
    }
    while let

以类似的方式, 当一个值匹配某种模式时，你可以用 while let 来进行条件循环。代码如下面所示:

loop {
match option {
Some(x) => println!("{}", x),
_ => break,
}
}

转换成下面这样的代码:

    while let Some(x) = option {
    println!("{}", x);
    }
    