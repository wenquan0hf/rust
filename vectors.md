#向量

一个‘向量’是一个动态的或者‘可增长的’数组，作为标准库类型 <a href="http://doc.rust-lang.org/stable/std/vec/">Vec<T\></a>来实现。**T** 表示我们可以有任何类型的向量（更多信息，请参照<a href="http://doc.rust-lang.org/stable/book/generics.html">泛型</a>的章节）。向量总是在堆上分配它们的数据。你可以使用 **vec!** 宏来创建它们：  
    
    let v = vec![1, 2, 3, 4, 5]; // v: Vec<i32>

（注意，与在之前我们使用的 **println!** 宏不同，对于 **vec!** 宏我们使用方括号 **[]**。Rust 允许您在两种情况下使用，这只是个约定。）  

对于重复一个初始值，这里有 **vec!** 的另一种形式：  
    
    let v = vec![0; 10]; // ten zeroes

##访问元素

若要获取在向量中的特定索引处的值，我们使用 **[]**：   
    
    let v = vec![1, 2, 3, 4, 5];
    
    println!("The third element of v is {}", v[2]);

由于指数从 **0** 开始，所以第三个元素是 **v[2]**。  

##循环访问 

一旦你有了一个向量，你可以通过 **for** 来遍历它的元素。这里有三个版本：   

    let mut v = vec![1, 2, 3, 4, 5];
    
    for i in &v {
    println!("A reference to {}", i);
    }
    
    for i in &mut v {
    println!("A mutable reference to {}", i);
    }
    
    for i in v {
    println!("Take ownership of the vector and its element {}", i);
    }

向量有许多更有用的方法，你可以在<a href="http://doc.rust-lang.org/stable/std/vec/">它们的 API 文档</a>中读到。
	
