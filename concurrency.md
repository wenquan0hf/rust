# 并发性

并发性和并行性在计算机科学中是非常重要的主题，即使在当今工业中也是个热门的话题。电脑得到了越来越多的核心，然而，很多程序并没有能力来利用它们。

Rust 内存安全特性同样采用了并发的方式。甚至 Rust 程序内存必须是安全，没有数据之间的竞争。Rust 的类型系统的任务就是给你强大的方式让程序能够在编译时并发执行。

之前我们谈论过 Rust 的并发特性，需要理解的重要的是: Rust 足够低级别的,所有这些都是由标准库提供的，而不是语言本身。这意味着，如果你不喜欢 Rust 某些方面处理并发性的方式，你可以自己实现另一种做事的方式。[mio](https://github.com/carllerche/mio) 是一个在现实中践行这一原则的例子。

## 后台：Send 和 Sync

并发性是很难说清楚的。在 Rust 中，我们有一个强大的、静态类型系统来帮助使我们的代码可理解。因此，Rust 提供给了我们两个特性来帮助我们理解可以并发执行的代码。

## Send

我们将谈论第一个特性是 Send。当类型 T 实现 Send 时，它告诉编译器，这种类型的线程拥有在线程之间安全转移的所有权。

强行添加一些限制是很重要的。例如，如果我们有一个通道连接两个线程，我们将希望能够向通道中发送一些数据，接着将这些数据传送给另外一个线程。因此，我们要保证要被发送的类型实现了 Send。

相反地，如果我们利用 FFI 封装一个库，然而它不是线程安全的，此时我们不想实现 Send，所以编译器会帮助我们强制它不能离开当前线程。

## Sync

第二个特性被称为 Sync。当一个类型 T 实现 Sync 时，它告诉编译器，但这种类型被使用在多个线程并发时不可能引起内存不安全的状态。

例如,与一个原子索引计数器共享不可变的数据是线程安全的。Rust 提供了一种类似 Arc< T > 类型，并且它实现了 Sync，所以它在线程之间共享是安全的。

这两个特性将会让你在并发的情况下，你所使用类型系统的对代码的属性做出强有力的保证。在说明为什么能够这样保障之前，我们需要首先学习如何创建一个并发的 Rust 程序。

## 线程

Rust 的标准库为线程提供了一个库，它允许你以并行的方式运行 Rust 代码。这里是使用 std::Thread 的一个简单例子:


```
use std::thread;

fn main() {
    thread::spawn(|| {
        println!("Hello from a thread!");
    });
}
```

thread:spawn() 方法接受一个封闭参数，这个参数会在一个新线程中执行。它返回该线程的句柄，它可以用来在等待子线程完成之后提取其结果:

```
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        "Hello from a thread!"
    });

    println!("{}", handle.join().unwrap());
}
```

许多语言有能力执行线程,但普遍的是非常不安全的。可以写整本书讲关于如何防止在共享可变状态的情况下发生错误。Rust 通过他的类型系统在编译期阻止数据的竞争来解决这个问题。接下来让我们看看如何在线程之间共享的数据。

## 安全共享可变状态 

由 Rust 的类型系统,我们可以有一个概念,尽管听起来不切实际:“安全共享可变状态。“许多程序员都同意共享可变状态是非常，非常糟糕的。

有人曾经说过：共享可变状态是一切罪恶的根源。大多数语言尝试解决这个问题通过“可变”部分,但 Rust 处理是通过解决“共享”的部分来解决这个问题。

同一所有权系统有助于防止错误的使用指针，同时也有助于排除数据竞争，这个是并发执行时最糟糕的一种 bug。

如下是一个 Rust 程序，它会在许多语言中存在数据竞赛。它将不会编译通过:


```
use std::thread;

fn main() {
    let mut data = vec![1u32, 2, 3];

    for i in 0..3 {
        thread::spawn(move || {
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

会输出如下的错误：

```
8:17 error: capture of moved value: `data`
        data[i] += 1;
        ^~~~
```

在这种情况下，我们知道我们的代码应该是安全的，但是 Rust 不确定。其实实际上它并不安全:如果我们在每个线程中都引用 data，那么每个线程都会有一个自己索引数据的权限，那么同一个数据有三个所有者！那是不好的。我们可以通过使用 `Arc<T>` 类型来解决这个问题，它是一个原子引用计数器数指针。“原子”意味着它跨线程共享是安全的。

`Arc< T >` 假定一个或更多关于其内容的属性，以确保它在跨线程共享是安全的：它假定其内容拥有  Sync 属性。但在我们的例子中，我们希望能够修改变量的值。我们需要一种类型能够确保一次只能有一个用户能够修改变量值。为此，我们可以使用  `Mutex< T >` 类型。这是第二个版本的代码。它仍然不能正常工作,但是由于其他的原因:

```
use std::thread;
use std::sync::Mutex;

fn main() {
    let mut data = Mutex::new(vec![1u32, 2, 3]);

    for i in 0..3 {
        let data = data.lock().unwrap();
        thread::spawn(move || {
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

这里存在如下的错误：

```
<anon>:9:9: 9:22 error: the trait `core::marker::Send` is not implemented for the type `std::sync::mutex::MutexGuard<'_, collections::vec::Vec<u32>>` [E0277]
<anon>:11         thread::spawn(move || {
                  ^~~~~~~~~~~~~
<anon>:9:9: 9:22 note: `std::sync::mutex::MutexGuard<'_, collections::vec::Vec<u32>>` cannot be sent between threads safely
<anon>:11         thread::spawn(move || {
                  ^~~~~~~~~~~~~
```

你可以发现，[Mutex](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html) 有一个 [lock](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html#method.lock) 方法，它的函数声明如下：
 
```
fn lock(&self) -> LockResult<MutexGuard<T>>
```

因为 Send 没有实现 `MutexGuard < T >`，我们不能越过线程范围进转换，这就是错误的原因。

我们可以用 `Arc< T >` 来解决这个问题。如下是可以工作的版本:

```
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(vec![1u32, 2, 3]));

    for i in 0..3 {
        let data = data.clone();
        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

现在我们可以在 Arc 中调用 clone() 方法，它会增加内部计数。这个句柄接着就会跳转到一个新的线程中进行执行。让我们更仔细地查看线程的主体代码：

```
thread::spawn(move || {
    let mut data = data.lock().unwrap();
    data[i] += 1;
});
```

首先，我们调用 lock() 函数获得互斥锁的锁。它将返回 `Result< T， E >`，而且这只是一个例子，由于这个函数可能会失败，因此我们使用 unwrap() 函数来得到引用的数据。真正的代码在这里会有更健壮的错误处理代码。当我们得到锁之后，我们就可以随意的修改变量了。

最后，在线程运行时，我们等待了一会。但这不是理想：我们可以选择一个合理的时间等待，但更有可能我们会是等待的时间比必要的时间长或者还要短，这取决于线程运行时执行计算花费的实际时间。

更精确的计时器的替代品是使用 Rust 标准库中提供的线程同步方法中的一个机制。接下来让我们谈谈其中一个的机制：通道。

## 通道 

如下是使用 channel 进行线程同步的一个版本，而不是等待一个特定的时间:

```
use std::sync::{Arc, Mutex};
use std::thread;
use std::sync::mpsc;

fn main() {
    let data = Arc::new(Mutex::new(0u32));

    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let (data, tx) = (data.clone(), tx.clone());

        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            *data += 1;

            tx.send(());
        });
    }

    for _ in 0..10 {
        rx.recv();
    }
}
```

我们使用 mpsc:channel() 方法来构造一个新的通道。我们只是发送一个简单的 () 到通道中，然后等待十次执行后返回。

虽然这通道只是发送一个通用的信号，但我们可以向通道中发送任何 Send 类型的数据！

```
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let tx = tx.clone();

        thread::spawn(move || {
            let answer = 42u32;

            tx.send(answer);
        });
    }

   rx.recv().ok().expect("Could not receive answer");
}
```

u32 是 Send 类型，因此我们可以复制。所以我们创建一个线程，让它来计算答案，然后使用 send() 通过通道向我们发送答案。

## 异常 

panic！ 将当前执行的线程中断。您可以用一个简单的隔离机制执行 Rust 的线程:

```
use std::thread;

let result = thread::spawn(move || {
    panic!("oops!");
}).join();

assert!(result.is_err());
```

线程返回给我们一个结果，这让我们能够检查线程是否发生了异常。
