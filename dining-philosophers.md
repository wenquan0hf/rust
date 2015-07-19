#哲学家就餐问题

对于我们的第二个项目，让我们来看一个典型的并发性问题。这就是“哲学家就餐问题”。这最初是由迪杰斯特拉在 1965 年提出的，但我们将要使用的版本出自托尼•霍尔在 1985 年发表的一篇[论文](http://www.usingcsp.com/cspbook.pdf)。

	在古代，一个富有的慈善家捐赠了一所学院来安排五个著名的哲学家。每个哲学家都有一个房间，他可以在其中从事他自己专业的思考活动；学校里有一个公共的餐厅，这里配备有一个圆形的桌子，桌子周边有五个椅子，每个椅子用坐在上面的哲学家的名字来标记。他们以逆时针的方向来围着桌子坐。每个哲学家的左侧放了一个金叉，桌子中间有一大碗不断添满的意大利面。哲学家将花费大部分时间去思考；但是当他饿了，他就会去餐厅，坐在自己的椅子上，拿起在自己左边的叉子来吃意大利面。但是意大利面条比较难吃到，需要第二个叉子才能把面条送到嘴里。因此，哲学家也要拿起他右边的叉子。当哲学家吃完面条后，他会把两个叉子都放下，离开自己的椅子，并继续思考。当然了，一个叉子一次只能有一个哲学家来使用。如果其他哲学家想要使用你的叉子，那么他仅仅需要等到这个叉子没人用了即可。

这个经典问题展示了一些不同的并发性的元素。原因在于，其实际实现起来比较复杂：一个简单的实现可能导致死锁。举一个例子，让我们先想一个简单的算法来解决这个问题：


1. 一个哲学家拿起了自己左边的叉子。
2. 然后他又拿起了他右边的叉子。
3. 吃意大利面。
4. 放下叉子。

现在，让我们来想象一下这一系列的事件：

1.	哲学家 1 开始此算法，拿起他左边的叉子。
2.	哲学家 2 开始此算法，拿起他左边的叉子。
3.	哲学家 3 开始此算法，拿起他左边的叉子。
4.	哲学家 4 开始此算法，拿起他左边的叉子。
5.	哲学家 5 开始此算法，拿起他左边的叉子。
6.	…？所有的餐叉都被拿起来了，没人能吃面了!

有不同的方法来解决这个问题。在本教程中我们将用我们的方法来解决它。现在，让我们开始从问题的本身开始建模。我们先从哲学家身上开始：

	struct Philosopher {
	    name: String,
	}
	
	impl Philosopher {
	    fn new(name: &str) -> Philosopher {
	        Philosopher {
	            name: name.to_string(),
	        }
	    }
	}
	
	fn main() {
	    let p1 = Philosopher::new("Baruch Spinoza");
	    let p2 = Philosopher::new("Gilles Deleuze");
	    let p3 = Philosopher::new("Karl Marx");
	    let p4 = Philosopher::new("Friedrich Nietzsche");
	    let p5 = Philosopher::new("Michel Foucault");
	}

在这里，我们创建了一个结构体来代表一个哲学家。在结构体里，我们现在所需要的仅仅是一个名字。我们将名字的类型选择成 String 型，而不是 &str。一般来说，使用一种自身拥有数据的类型比使用一个利用引用的类型容易的多。

我们继续看下面的部分：

	impl Philosopher {
	    fn new(name: &str) -> Philosopher {
	        Philosopher {
	            name: name.to_string(),
	        }
	    }
	}

这里的 impl 块能让我们为 Philosopher 结构体定义一些内容。在本例中，我们定义了一个名叫 new 的关联函数。它的第一行如下所示：

	fn new(name: &str) -> Philosopher {

这里我们需要输入一个参数 name，类型是 &str。这是另一个字符串的一个引用。此函数会返回一个 Philosopher 结构体的实例。

	Philosopher {
	    name: name.to_string(),
	}

这将会创建一个新的 Philosopher，并将它的 name 值设置成前面 name 的参数值。但这不是参数的自身，因为我们对其调用了 .to\_string()。这将创建一个 &str 所指向字符串的副本，此副本的类型是Philosopher 的 name 字段的 String 类型。

那么为什么不直接用 String 类型的参数？这样更容易调用。如果我们采用了 String 类型参数，但是我们的调用者所有的是 &str，那么他们必须自己调用这个 .to\_string() 的方法。这种灵活性的缺点是，它总是产生一个副本。对于这个小程序，这并不是特别重要，因为我们知道，我们仅仅使用的是短字符串。

最后一件你可能会注意到的事：我们只定义了一个 Philosopher，似乎用它什么事情也没有做。Rust  是一种基于表达式的语言，这意味着 Rust 中几乎所有的内容都是返回一个值的表达式。这个说法对函数亦是如此，因为函数的最后一个表达式将自动返回。由于这个函数的最后一个表达式是我们创建了一个新的 Philosopher，我们最终返回这个 Philosopher。

对于 Rust，这个名字，new()，没有什么特别，但是它是创建新的结构体实例的函数的约定俗成的叫法。在我们讨论为什么是这样之前，让我们先再看看 main()：

	fn main() {
	    let p1 = Philosopher::new("Baruch Spinoza");
	    let p2 = Philosopher::new("Gilles Deleuze");
	    let p3 = Philosopher::new("Karl Marx");
	    let p4 = Philosopher::new("Friedrich Nietzsche");
	    let p5 = Philosopher::new("Michel Foucault");
	}

在这里，我们创建了五个 Philosopher 的变量绑定。这五个名字都是我最喜欢的，你也可以用任何你想要的名字来替换他们。如果我们*没有*定义 new() 函数，它会看起来会是这样的：

	fn main() {
	    let p1 = Philosopher { name: "Baruch Spinoza".to_string() };
	    let p2 = Philosopher { name: "Gilles Deleuze".to_string() };
	    let p3 = Philosopher { name: "Karl Marx".to_string() };
	    let p4 = Philosopher { name: "Friedrich Nietzche".to_string() };
	    let p5 = Philosopher { name: "Michel Foucault".to_string() };
	}

这看起来就比较麻烦了。当然使用 new 也有其他的一些优势，而在我这个简单的例子中，使用它也使我们的代码简洁了不少。

现在我们在这里已经写好了基本的内容，同时会有很多方法可以解决上述问题。我想从问题的结束端先开始：让我们建立一个每个哲学家吃完的方案。这是很小的一步，我们先创建一个方法，然后遍历所有的哲学家，如下所示：

	struct Philosopher {
	    name: String,
	}   
	
	impl Philosopher { 
	    fn new(name: &str) -> Philosopher {
	        Philosopher {
	            name: name.to_string(),
	        }
	    }
	    
	    fn eat(&self) {
	        println!("{} is done eating.", self.name);
	    }
	}
	
	fn main() {
	    let philosophers = vec![
	        Philosopher::new("Baruch Spinoza"),
	        Philosopher::new("Gilles Deleuze"),
	        Philosopher::new("Karl Marx"),
	        Philosopher::new("Friedrich Nietzsche"),
	        Philosopher::new("Michel Foucault"),
	    ];
	
	    for p in &philosophers {
	        p.eat();
	    }
	}

让我们首先来看 main()。我们没有采用有五个哲学家 Philosopher 的变量绑定，相反我们创建了一个哲学家的 Vec<T>。 Vec<T> 也被称为“向量”，这是一个可增长的数组类型。然后，我们使用一个 for 循环迭代向量体，也就是会轮流的获取到每个哲学家的引用。

在循环体内，我们调用的是 p.eat()，下面是关于其函数定义：

	fn eat(&self) {
	    println!("{} is done eating.", self.name);
	}

在Rust中，方法可以用一个明确的 self 参数。这就是为什么 eat() 是一个方法，而 new 是一个关联函数： new() 中没有 self。我们 eat() 的第一个版本，只打印出哲学家的名字，并且提及他们正在吃。现在运行这个程序，你可以得到以下的输出：

	Baruch Spinoza is done eating.
	Gilles Deleuze is done eating.
	Karl Marx is done eating.
	Friedrich Nietzsche is done eating.
	Michel Foucault is done eating.

够简单吧，程序运行完全没有问题!然而我们还没有实现真正的问题，所以我们还没有完成整个程序!

接下来，我们想想做的是：我们的哲学家不仅吃完，而且实际上他们也是在吃的。下面是程序的下一个版本：

	use std::thread;
	
	struct Philosopher {
	    name: String,
	}   
	
	impl Philosopher { 
	    fn new(name: &str) -> Philosopher {
	        Philosopher {
	            name: name.to_string(),
	        }
	    }
	    
	    fn eat(&self) {
	        println!("{} is eating.", self.name);
	
	        thread::sleep_ms(1000);
	
	        println!("{} is done eating.", self.name);
	    }
	}
	
	fn main() {
	    let philosophers = vec![
	        Philosopher::new("Baruch Spinoza"),
	        Philosopher::new("Gilles Deleuze"),
	        Philosopher::new("Karl Marx"),
	        Philosopher::new("Friedrich Nietzsche"),
	        Philosopher::new("Michel Foucault"),
	    ];
	
	    for p in &philosophers {
	        p.eat();
	    }
	}

仅有几个变化。让我们一个一个的来讲。

	use std::thread;

use 可以使后面的库加载在作用域内。我们稍后会使用标准库中的 thread 模块，所以我们需要 use 它。

    fn eat(&self) {
        println!("{} is eating.", self.name);

        thread::sleep_ms(1000);

        println!("{} is done eating.", self.name);
    }

我们现在打印了两个消息，其中间有一个 sleep\_ms() 将其分隔开。sleep\_ms() 将模拟哲学家吃的时间。

如果现在你运行这个程序，你会看到每个哲学家轮流吃：

	Baruch Spinoza is eating.
	Baruch Spinoza is done eating.
	Gilles Deleuze is eating.
	Gilles Deleuze is done eating.
	Karl Marx is eating.
	Karl Marx is done eating.
	Friedrich Nietzsche is eating.
	Friedrich Nietzsche is done eating.
	Michel Foucault is eating.
	Michel Foucault is done eating.

太好了!我们又前进了一步。这里还有一个问题：我们实际上并没有以并行的方式操作，并行才是就餐问题的核心部分!

为了能让我们的哲学家并发的吃，我们需要做一个小改变。下面是下一个版本：

	use std::thread;
	
	struct Philosopher {
	    name: String,
	}   
	
	impl Philosopher { 
	    fn new(name: &str) -> Philosopher {
	        Philosopher {
	            name: name.to_string(),
	        }
	    }
	
	    fn eat(&self) {
	        println!("{} is eating.", self.name);
	
	        thread::sleep_ms(1000);
	
	        println!("{} is done eating.", self.name);
	    }
	}
	
	fn main() {
	    let philosophers = vec![
	        Philosopher::new("Baruch Spinoza"),
	        Philosopher::new("Gilles Deleuze"),
	        Philosopher::new("Karl Marx"),
	        Philosopher::new("Friedrich Nietzsche"),
	        Philosopher::new("Michel Foucault"),
	    ];
	
	    let handles: Vec<_> = philosophers.into_iter().map(|p| {
	        thread::spawn(move || {
	            p.eat();
	        })
	    }).collect();
	
	    for h in handles {
	        h.join().unwrap();
	    }
	}

我们所要做的就是改变 main() 里面的循环，在其中添加上第二个循环体!下面是第一个改变：

	let handles: Vec<_> = philosophers.into_iter().map(|p| {
	    thread::spawn(move || {
	        p.eat();
	    })
	}).collect();

虽然这里只有五行，但这五行代码确很密集。让我们一个一个的来讲。

	let handles: Vec<_> =

我们这里引入了一个名叫 handles 新的绑定。我们这样叫它，是因为我们要创建一些新的线程，而这些线程会返回一些能让我们控制此些线程操作的 handles。由于一个我们过后要讲到的问题，这里我们要明确解释这个类型。\_ 是一个类型的占位符。我们说“handles 是一个某项的向量，在Rust 中，你可以找出到这项到底是什么。”

	philosophers.into_iter().map(|p| {

我们的哲学家 philosophers 调用了 into_iter()。这将创建一个迭代器，它将拥有每个哲学家的所有权。我们要这样才能将哲学家传递到线程中。我们用迭代器调用 map，它以一个闭包作为参数，并且依次调用闭包中的每个元素。

    thread::spawn(move || {
        p.eat();
    })

这里就是发生并发的地方。thread：：spawn 函数以一个闭包作为参数，并会在一个新线程里执行这个闭包。关于这个闭包的一些额外的解释， move，表明闭包要获取它的捕获值的所有权。主体函数上，p 是 map 函数的变量。

在线程内部，我们所做的就是用 p 调用 eat()。

	}).collect();

最后，我们获取了所有的 map 调用的结果并把它们收集起来。collect() 会将它们生成某种集合，这就是为什么我们要解释返回类型的原因：因为我们想要一个 Vec<T>。集合的元素就是 thread：：spawn 调用的返回值，也就是这些线程的 handles。唷! 

	for h in handles {
	    h.join().unwrap();
	}

main() 函数的结尾部分，我们对 handles 进行循环处理，对其调用 join()，这可以阻塞其它线程的执行，直到本线程执行完成。这确保了线程在程序将其退出前会完成它们的工作。

如果你运行这个程序，你会发现哲学家可以自由的吃啦!到这儿，我们已经有多线程程序啦!

	Gilles Deleuze is eating.
	Gilles Deleuze is done eating.
	Friedrich Nietzsche is eating.
	Friedrich Nietzsche is done eating.
	Michel Foucault is eating.
	Baruch Spinoza is eating.
	Baruch Spinoza is done eating.
	Karl Marx is eating.
	Karl Marx is done eating.
	Michel Foucault is done eating.

但是叉子呢？我们还没有为它们建模。

要做到这一点，让我们写一个新的 struct：

	use std::sync::Mutex;
	
	struct Table {
	    forks: Vec<Mutex<()>>,
	}

这桌子 Table 有一个互斥元(Mutex)的向量。互斥元是控制并发性的一种方法：一次只能有一个线程可以访问内容。这正是我们叉子所需要的属性。在互斥元中我们用了一个空的元组，()，因为我们并不打算使用其中的值，仅仅获取到它就可以。

让我们使用 Table 来修改这个程序：

	use std::thread;
	use std::sync::{Mutex, Arc};
	
	struct Philosopher {
	    name: String,
	    left: usize,
	    right: usize,
	}
	
	impl Philosopher {
	    fn new(name: &str, left: usize, right: usize) -> Philosopher {
	        Philosopher {
	            name: name.to_string(),
	            left: left,
	            right: right,
	        }
	    }
	
	    fn eat(&self, table: &Table) {
	        let _left = table.forks[self.left].lock().unwrap();
	        let _right = table.forks[self.right].lock().unwrap();
	
	        println!("{} is eating.", self.name);
	
	        thread::sleep_ms(1000);
	
	        println!("{} is done eating.", self.name);
	    }
	}
	
	struct Table {
	    forks: Vec<Mutex<()>>,
	}
	
	fn main() {
	    let table = Arc::new(Table { forks: vec![
	        Mutex::new(()),
	        Mutex::new(()),
	        Mutex::new(()),
	        Mutex::new(()),
	        Mutex::new(()),
	    ]});
	
	    let philosophers = vec![
	        Philosopher::new("Baruch Spinoza", 0, 1),
	        Philosopher::new("Gilles Deleuze", 1, 2),
	        Philosopher::new("Karl Marx", 2, 3),
	        Philosopher::new("Friedrich Nietzsche", 3, 4),
	        Philosopher::new("Michel Foucault", 0, 4),
	    ];
	
	    let handles: Vec<_> = philosophers.into_iter().map(|p| {
	        let table = table.clone();
	
	        thread::spawn(move || {
	            p.eat(&table);
	        })
	    }).collect();
	
	    for h in handles {
	        h.join().unwrap();
	    }
	}

又有了许多的变化!这一次我们有了最终可运行的版本。我们看下细节内容：

	use std::sync::{Mutex, Arc};

我们将使用 std：：sync 包中的另一个结构：Arc<T>。当我们使用它时，会做进一步的讲解。

	struct Philosopher {
	    name: String,
	    left: usize,
	    right: usize,
	}

我们需要为 Philosopher 添加两个字段。每个哲学家都有两个叉子：一个在左，一个在右。我们用 usize 类型来表示它们，因为这类型是索引向量。这两个值将被索引到我们 Table 中的 forks 了。

	fn new(name: &str, left: usize, right: usize) -> Philosopher {
	    Philosopher {
	        name: name.to_string(),
	        left: left,
	        right: right,
	    }
	}

我们现在需要构造 left 和 right 的值了，所以我们将它们添加到了 new() 中。

	fn eat(&self, table: &Table) {
	    let _left = table.forks[self.left].lock().unwrap();
	    let _right = table.forks[self.right].lock().unwrap();
	
	    println!("{} is eating.", self.name);
	
	    thread::sleep_ms(1000);
	
	    println!("{} is done eating.", self.name);
	}

我们又添加了两行新代码。我们还添加了一个参数，table。这样我们就可以访问 Table 的 fork 列表，然后可以用 self.left 和 self.right 访问特定索引上的 fork。让我们可以访问那个索引上的互斥元 Mutex，并且用其调用 lock()。如果互斥元目前正在被别人访问，那么我们的线程将被阻塞，直到互斥元变得可用。

lock() 的调用可能会失败，如果是这样，程序会崩溃。在这种情况下，错误的发生可能是因为互斥元“[中毒](https://doc.rust-lang.org/stable/std/sync/struct.Mutex.html#poisoning)”了，这是由于当锁已经被持有时会引起线程应急。因为这样的错误不该发生，所以我们使用 unwrap() 来解决它。

这些行另一个奇怪的现象是：我们将结果命名为 \_left 和 \_right。这些下划线有什么用？嗯，因为我们不打算使用锁内的值。我们只是想持有锁。因此，Rust 会警告我们，我们从来没有使用过锁内值。通过使用下划线，我们就会告诉 Rust，这就是我们想要的，这样的话 Rust 就不会抛出警告。

那么如何释放锁呢？嗯，当 _left 和 _right 超出作用域后，锁会自动释放。

    let table = Arc::new(Table { forks: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});

接下来，在 main() 中，我们创建了一个新的 Table 并定义成  Arc<T>  类型。“arc” 是 “atomic reference count” 的缩写，我们要通过多线程来共享我们的 Table。当我们要共享它时，引用数就会上升，当每个线程结束时，引用数就会减少。

	let philosophers = vec![
	    Philosopher::new("Baruch Spinoza", 0, 1),
	    Philosopher::new("Gilles Deleuze", 1, 2),
	    Philosopher::new("Karl Marx", 2, 3),
	    Philosopher::new("Friedrich Nietzsche", 3, 4),
	    Philosopher::new("Michel Foucault", 0, 4),
	];

我们要将我们的 left 和 right 值传递到 Philosopher 的构造函数中。这里有一个非常重要的细节。如果你查看他们的样式，开始直到最后都是一致的。Monsieur Foucault 本应该用 4，0 作为参数，但相反的，用 0，4 作了参数。这是其实是防止死锁：我们的一个哲学家是左撇子!这是解决问题的一种方法，在我看来，这是最简单的。

	let handles: Vec<_> = philosophers.into_iter().map(|p| {
	    let table = table.clone();
	
	    thread::spawn(move || {
	        p.eat(&table);
	    })
	}).collect();

最后，在 map()/collect() 循环中，我们调用了 table.clone()。Arc<T> 调用的 clone() 方法会增加引用的计数，但是当它超出作用域之后，引用的计数会相应的减少。你会注意到，我们可以在这里引入一个新的 table 的绑定，它将覆盖旧的那一个。这个方法经常使用。这样可以使你不需要命名两个唯一的名字。

到这里，我们的程序就可以工作啦!只有两个哲学家可以在同一时间吃，所以你会得到一些如下的输出：

	Gilles Deleuze is eating.
	Friedrich Nietzsche is eating.
	Friedrich Nietzsche is done eating.
	Gilles Deleuze is done eating.
	Baruch Spinoza is eating.
	Karl Marx is eating.
	Baruch Spinoza is done eating.
	Michel Foucault is eating.
	Karl Marx is done eating.
	Michel Foucault is done eating.

恭喜你!你已经用 Rust 实现了一个典型的并发性问题。

