# 基准测试 #
Rust支持基准测试来测试用户代码的性能。我们来看一下 `src/lib.rs` 的性能如何。


	`#![feature(test)]

	extern crate test;

	pub fn add_two(a: i32) -> i32 {
 	   a + 2
	}

	#[cfg(test)]
	mod tests {
    use super::*;
    use test::Bencher;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }

    #[bench]
    fn bench_add_two(b: &mut Bencher) {
        b.iter(|| add_two(2));
    }
	}`


注意上面代码中的 `test` 的功能声明，这表示这并不是一个稳定的功能。

用户需要引入 `test` 的封装，来使得基准测试得以支持。通过 `bench` 属性的使用，我们可以实现一个新的函数。与不能有参数的常规测试不同，这里的基准测试使用 `&mut Bencher` 来改善这个情况。这里的 `Bencher` 提供了一个闭包的 iter 方法。这个闭包就包含了需要进行基准测试的代码。

用户通过 `cargo bench` 指令来执行基准测试：

    $ cargo bench
       Compiling adder v0.0.1 (file:///home/steve/tmp/adder)
     Running target/release/adder-91b3e234d4ed382a
    
    running 2 tests
    test tests::it_works ... ignored
    test tests::bench_add_two ... bench: 1 ns/iter (+/- 0)
    
    test result: ok. 0 passed; 0 failed; 1 ignored; 1 measured

我们的非基准测试可以被忽略掉。用户也许会注意到  `cargo bench` 比 `cargo test` 长一个比特。这是因为 Rust 会多次执行基准测试，然后会取一个平均值。因为在这个例子中，只是实现了一些简单的功能，所以我们有一个 `ns/iter (+/- 0)`，但是这会展示出结果的方差。

如下是编写基准测试的建议：

1. 将安装代码移到 `iter` 循环之外，只是将用户希望进行测试的代码放到 `iter` 循环内。
1. 将实现同一功能的代码放到迭代体内，不要将结果进行累计，也不要改变状态。
1. 将外部函数进行幂等化，基准测试程序会将它测试多次。
1. 尽量使 `iter` 循环体内容的代码更精简，然后使得基准测试运行起来更快捷，使得校准器可以以更高精度的来校准。
1. 使 `iter` 循环体内的代码更简单，以帮助查明性能改进的地方（或回归）。


## 疑难杂症：优化 ##
在基准测试程序的编写方面还存在一些辣手的问题：优化编译后的基准测试代码可能会被优化器篡改，这样使得基准测试可能就不再是用户希望的测试对象了。比如，编译器可能会重新组织一些代码，因为这些代码并没有什么作用，甚至会将它全部删除。


    #![feature(test)]

	extern crate test;
	use test::Bencher;

	#[bench]
	fn bench_xor_1000_ints(b: &mut Bencher) {
	b.iter(|| {
	(0..1000).fold(0, |old, new| old ^ new);
	});
	}
    



上述代码会出现下述结果：

    running 1 test
    test bench_xor_1000_ints ... bench: 0 ns/iter (+/- 0)
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured


为了避免上述问题，基准测试的执行者会采用两个方法。`iter` 方法可以获得一个闭包，它可以返回一个任意值来迫使优化器考虑这个结果，来保证优化器不会更改基准测试代码。下面的例子就是来展示如何校准 `b.iter`.

	`b.iter(|| {  
    // note lack of `;` (could also use an explicit `return`).
    (0..1000).fold(0, |old, new| old ^ new)
	});`

另一种方法是通用的 `test::black_box` 函数，它对优化器就是一个“黑盒”，可以迫使优化器考虑更多的参数。

    #![feature(test)]
    
    extern crate test;
    
    b.iter(|| {
    let n = test::black_box(1000);
    
    (0..n).fold(0, |a, b| a ^ b)
    })
    


这些并不会读取和更改这个值。较大的值可以直接降低开销。

	`black_box(&huge_struct)`

执行上述任何一种变化提供了以下基准测试结果。


    running 1 test
    test bench_xor_1000_ints ... bench:   131 ns/iter (+/- 3)
    
    test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured


即使使用上述方式,优化器仍然会以不可想象的方式来修改测试用例。

