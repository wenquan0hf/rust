##编译器插件

###简介

rustc 可以加载编译器插件，它是用户提供的库,这个库使用新语法扩展编译器的行为，lint 检查等。　　　　

插件是一个有指定的 registrar 函数的动态库，注册 rustc 扩展。其它库可以使用属性 #![plugin(...)] 加载这些扩展。想了解更多关于定义机制和加载插件，查看 rustc::plugin 文档。

如果存在，像 #![plugin(foo(... args ...))] 传递的参数不被 rustc 本身编译。他们通过注册表参数的方法提供给插件。　　　　

在绝大多数情况下，一个插件只能通过 #![plugin] 而不是通过一个 extern crate 项目来被使用。连接一个插件将会把所有 libsyntax 和 librustc 作为库的依赖关系。这通常是不必要的，除非你正在创建另一个插件。plugin_as_library lint检查这些准则。　　　　

通常的做法是将编译器插件放入自己的库中，独立于库中被客户端使用的任何 macro_rules ! 宏或普通 Rust 代码。

###语法扩展

插件可以用不同的方法扩展 Rust 的语法。一种语法扩展是程序宏。调用程序宏就像调用普通的宏一样，但扩展是由任意在运行时操纵语法树的 Rust 代码执行。　　　　

让我们写一个插件 roman_numerals.rs 实现罗马数字整数常量。

    #![crate_type="dylib"]
    #![feature(plugin_registrar, rustc_private)]
    
    extern crate syntax;
    extern crate rustc;
    
    use syntax::codemap::Span;
    use syntax::parse::token;
    use syntax::ast::{TokenTree, TtToken};
    use syntax::ext::base::{ExtCtxt, MacResult, DummyResult, MacEager};
    use syntax::ext::build::AstBuilder;  // trait for expr_usize
    use rustc::plugin::Registry;
    
    fn expand_rn(cx: &mut ExtCtxt, sp: Span, args: &[TokenTree])
    -> Box<MacResult + 'static> {
    
    static NUMERALS: &'static [(&'static str, u32)] = &[
    ("M", 1000), ("CM", 900), ("D", 500), ("CD", 400),
    ("C",  100), ("XC",  90), ("L",  50), ("XL",  40),
    ("X",   10), ("IX",   9), ("V",   5), ("IV",   4),
    ("I",1)];
    
    let text = match args {
    [TtToken(_, token::Ident(s, _))] => token::get_ident(s).to_string(),
    _ => {
    cx.span_err(sp, "argument should be a single identifier");
    return DummyResult::any(sp);
    }
    };
    
    let mut text = &*text;
    let mut total = 0;
    while !text.is_empty() {
    match NUMERALS.iter().find(|&&(rn, _)| text.starts_with(rn)) {
    Some(&(rn, val)) => {
    total += val;
    text = &text[rn.len()..];
    }
    None => {
    cx.span_err(sp, "invalid Roman numeral");
    return DummyResult::any(sp);
    }
    }
    }
    
    MacEager::expr(cx.expr_u32(sp, total))
    }
    
    #[plugin_registrar]
    pub fn plugin_registrar(reg: &mut Registry) {
    reg.register_macro("rn", expand_rn);
    }

然后我们可以像使用其他宏一样使用 rn !()：

    #![feature(plugin)]
    #![plugin(roman_numerals)]
    
    fn main() {
    assert_eq!(rn!(MMXV), 2015);
    }

一个简单的 fn(&str) -> u32 的优势是：

- (任意复杂的)转换是在编译时完成的。　　
- 输入验证在编译时执行。　　
- 它可以扩展到模式中允许使用，它有效地给出了一个为数据类型定义新的文字语法的方法。

除了程序宏，你可以定义新的 derive-like 属性和其他类型的扩展。请看Registry::register_syntax_extension 和 SyntaxExtension enum。更多调用宏的例子，请见 regex_macros。

###提示和技巧

有一些宏的调试技巧是适用的。　　　　

您可以使用 syntax::parse 将标记树转化为更高级的语法元素如表达式：

    fn expand_foo(cx: &mut ExtCtxt, sp: Span, args: &[TokenTree])
    -> Box<MacResult+'static> {
    
    let mut parser = cx.new_parser_from_tts(args);
    
    let expr: P<Expr> = parser.parse_expr();

通过这些 libsyntax 解析代码我们可以知道解析基础结构是如何工作的。　　　　

为得到更准确的错误报告，保持所有你解析的代码的 span 。你可以将 spaned 封装到自定义数据结构中。　　　　

调用 ExtCtxt:span_fatal 会立即中止编译。最好不要调用 ExtCtxt:span_err 并返回 DummyResult，编译器可以继续并找到更多的错误。　　　　

为了打印语法片段进行调试，可以使用 span\_note 加上syntax::print::pprust::*_to_string。

上面的例子使用 AstBuilder::expr_usize 产生一个整数。除了 AstBuilder 特征，libsyntax 提供了一组 quasiquote 宏。他们没有正式文件并且非常粗糙的。然而，它的实现可能是一个改进的一个普通的插件库 quasiquote 的好的起点 。

###Lint 插件

插件可以通过对额外的代码类型、安全等等的检查来扩展 Rust 的 lint 基础结构。在 src/test/auxiliary/lint_plugin\_test.rs 中你可以看到一个完整的例子。这个例子的核心如下：

    declare_lint!(TEST_LINT, Warn,
      "Warn about items named 'lintme'");
    
    struct Pass;
    
    impl LintPass for Pass {
    fn get_lints(&self) -> LintArray {
    lint_array!(TEST_LINT)
    }
    
    fn check_item(&mut self, cx: &Context, it: &ast::Item) {
    let name = token::get_ident(it.ident);
    if name.get() == "lintme" {
    cx.span_lint(TEST_LINT, it.span, "item is named 'lintme'");
    }
    }
    }
    
    #[plugin_registrar]
    pub fn plugin_registrar(reg: &mut Registry) {
    reg.register_lint_pass(box Pass as LintPassObject);
    }

然后代码

    #![plugin(lint_plugin_test)]
    
    fn lintme() { }

将会产生一个编译器警告：

    foo.rs:4:1: 4:16 warning: item is named 'lintme', #[warn(test_lint)] on by default
    foo.rs:4 fn lintme() { }
     ^~~~~~~~~~~~~~~

lint插件的组件如下：
 
- 一个或多个 定义静态的 Lint 结构的 declare_lint ! 调用；　　　
- 一个控制 int pass (here, none) 所需的任何 state 的 struct；
- 一个定义如何检查每个语法元素的 LintPass 实现。一个 LintPass 可能为几个不同的 Lint 调用span_lint，但应该通过 get_lints 方法注册它们。　　　　　　

Lint 通过遍历语法，但他们在编译的后期运行，在哪里可以获得类型信息。rustc 内置的 lint 大多使用相同的基础结构作为 lint 插件，并提供了一些说明如何访问类型信息的例子。

插件定义的 lint 是由通常的属性和编译器标志所控制，例如 #[allow(test\_lint)] 或 -A test-lint。通过合适的案例和标点符号的转换，这些标识符由第一个参数传递到 declare_lint !。

您可以运行 rustc -W help foo.rs 来看 rustc 知道的 lint 的列表，包括那些由foo.rs 加载的插件提供的列表。