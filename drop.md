# 降

既然我们已经讨论了特征，让我们谈谈 Rust 标准库提供的特定特征，降降特性提供了一种方法，这种方法可以保证即便某个值超出范围时也能运行代码。例如：

    struct HasDrop;
    
    impl Drop for HasDrop {
    fn drop(&mut self) {
    println!("Dropping!");
    }
    }
    
    fn main() {
    let x = HasDrop;
    
    // do stuff
    
    } // x goes out of scope here

当 x 在 main()函数结尾超出范围时，降的代码将运行。降有一个方法,该方法也被称为 drop()。它将 self 作为可变参考。

这正是我们想要的!降的机制是非常简单的，但是有一些微妙之处。例如，值以与它的声明相反的顺序下降。这里还有一个例子：

    struct Firework {
    strength: i32,
    }
    
    impl Drop for Firework {
    fn drop(&mut self) {
    println!("BOOM times {}!!!", self.strength);
    }
    }
    
    fn main() {
    let firecracker = Firework { strength: 1 };
    let tnt = Firework { strength: 100 };
    }

这将输出：

```
BOOM times 100!!!

BOOM times 1!!!
```

TNT 在爆竹之前爆炸，因为它是之后被声明的。后进先出。

那降有什么优点呢？一般来说,降是用来清理任何与结构相关联的资源。例如，`ARC<T> TYPE` 类型是一个引用计数的类型。调用降时，它将减量引用计数，如果引用的总数为零，它将清理潜在值。
