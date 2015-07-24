# 切片模式

如果用户希望匹配一个切片或数组，用户可以使用 `&` 来修饰 `slice_patterns` 功能。

    #![feature(slice_patterns)]
    
    fn main() {
    let v = vec!["match_this", "1"];
    
    match &v[..] {
    ["match_this", second] => println!("The second element is {}", second),
    _ => {},
    }
    }

`advanced_slice_pattern` 使用户可以使用 `..` 来表示切片匹配模式内部的元素任何数目。此通配符仅能为给定的数组使用一次。如果在 `..` 前有个标识符，切片的结果将会绑定到这个名字。比如:

	`#![feature(advanced_slice_patterns, slice_patterns)]

	fn is_symmetric(list: &[u32]) -> bool {
	    match list {
	        [] | [_] => true,
	        [x, inside.., y] if x == y => is_symmetric(inside),
	        _ => false
	    }
	}

	fn main() {
    let sym = &[0, 1, 4, 2, 4, 1, 0];
    assert!(is_symmetric(sym));

    let not_sym = &[0, 1, 7, 2, 4, 1, 0];
    assert!(!is_symmetric(not_sym));
	} `