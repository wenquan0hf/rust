##Nightly Rust

Rust 提供了三个版本渠道：nightly，beta，还有stable。不稳定特性只在 Nightly Rust 有效。这个过程的更多细节，请参见 “Stability as a deliverable”。　　　　

安装 nightly Rust，你可以使用 rustup.sh：

    $ curl -s https://static.rust-lang.org/rustup.sh | sh -s -- --channel=nightly

如果你担心使用 curl | sh 有潜在的不安全性，请阅读下面我们的免责声明。免费使用两个版本的安装和检查我们的安装脚本：

    $ curl -f -L https://static.rust-lang.org/rustup.sh -O
    $ sh rustup.sh --channel=nightly

如果你在使用 Windows 系统，请下载32位的安装程序或64位安装程序并运行它。

###卸载

如果你决定你不在使用 Rust 了，我们会有点难过，但没关系。并不是每一个编程语言对每个人来说都是很好用的。只需运行卸载脚本：

    $ sudo /usr/local/lib/rustlib/uninstall.sh

如果你使用 Windows 安装程序，只要重新运行 .msi，它会给你一个卸载选项。

理所当然，当我们告诉你 curl | sh，有些人变得非常沮丧。基本上，当你这样做时，你相信维护 Rust 的人不会攻击你的电脑和做其它坏事。这是一个很好的本能！如果你是这些人中的一员，请查看从源代码构建 Rust 的文档，或官方二进制下载。

哦，我们还应该提到官方支持平台：

- Windows (7， 8， Server 2008 R2)
- Linux (2.6.18 或更新版本， 或不同版本)， x86 和 x86-64
- OSX 10.7 (Lion) 或更高版本， x86 和 x86-64

我们在这些平台上广泛测试 Rust，还有其他平台，像 Android。但这些都是最有可能的运行平台，因为他们都是最经常测试的平台。　　　　

最后，看一下 Windows 。Rust 认为 Windows 是发布的一个一流平台，但如实说来，Windows 体验不像 Linux / OS X 体验一样集成。我们正在努力做到这一点! 如果有什么不起作用，这就是一个漏洞。请让我们看看这是否真的发生。测试对 Windows 的每个提交就像测试任何其他平台一样。　　　　

如果你已经安装 Rust ，你可以打开一个 shell 和类型：

    $ rustc --version

你应该看到版本号、提交哈希表，提交日期和构建日期：

    rustc 1.0.0-nightly (f11f3e7ba 2015-01-04) (built 2015-01-06)

如果你这么做了，Rust 已经安装成功！恭喜！

这个安装程序也在本地安装一个文档的副本，这样你就可以离线阅读。在 UNIX 系统中，/usr/local/share/doc/rust 就是安装地址。在 Windows 中，安装在一个 share/doc 目录中，无论你将 Rust 安装在何处。　　　　

如果没有安装，有很多地方你可以得到帮助。最简单的就是 the #rust IRC channel on irc.mozilla.org ，你可以通过 Mibbit 访问到。单击该链接，你可以与其他 Rustaceans 交流，我们可以帮助你。还有其它的资源，包括用户的论坛和堆栈溢出。