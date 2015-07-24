# 安装 Rust

使用 Rust 的第一步当然是安装它，有许多方法来安装 Rust，但其中最简单的方法是使用 rustup 脚本。如果你使用的是 Linux 或 Mac，所有你需要做的就是这些(请注意，你不需要输入 $，它们只显示每个命令的开始)：

```
	$ curl -sf -L https://static.rust-lang.org/rustup.sh | sh
```

如果你担心使用 `curl | sh` 所潜在的不安全，请继续往下看我们的免责声明。也可自由的选择使用两步的安装版本-安装和检查的脚本：

```
	$ curl -f -L https://static.rust-lang.org/rustup.sh -O
	$ sh rustup.sh
```

如果你使用的是 Windows，请下载适合版本的[安装程序](http://www.rust-lang.org/install.html)。

## 卸载

如果你决定不再想使用 Rust，我们会有点难过，不过没关系。并不是每一个编程语言对每个人来说都是合适的。运行下面的卸载脚本即可：

```
 	$ sudo /usr/local/lib/rustlib/uninstall.sh
```

如果你使用的是 Windows 安装程序，重新运行 .msi 文件,它会显示给你一个卸载选项。

当我们告诉你使用 curl | sh 时，有些人会觉得非常不爽，某种程度上来说这是自然而然的。基本上当你这样做时,你是相信维护 Rust 的人是好人，并且他们不会攻击你的电脑或者做坏事的。这是一个很好的本能!如果你是这些人中的一员，请查看文档[从源代码构建 Rust](https://github.com/rust-lang/rust#building-from-source) ，或[官方下载](http://www.rust-lang.org/install.html).

哦,我们还应该提到官方支持的平台：

- Windows (7, 8, Server 2008 R2)
- Linux (2.6.18 or later, various distributions), x86 and x86-64
- OSX 10.7 (Lion) or greater, x86 and x86-64

我们在这些平台上对 Rust 进行了大量的测试，像其他平台，如 Android，也做了一些测试，但是推荐利用上述几个平台，因为他们有最多的测试。

最后，关于 Windows 的一个说明。Rust 认为 Windows 是一个一流的发布平台，但我们诚实的讲，Windows 的体验不如 Linux/OS X 体验集成度好。我们正在努力提高它！如果有什么导致 Rust 不能工作，那这就是一个错误。如果这真的发生了，请通知我们。每一个错误的提交都会向任何其他平台一样，会在 Windows 上测试。

如果你已经安装了 Rust，你可以打开一个 shell，输入以下命令：

```
	$ rustc --version
```

你将会看到版本号、提交哈希和提交日期。如果你只是安装版本1.0.0，你将会看到：

```
	rustc 1.0.0 (a59de37e9 2015-05-13)
```

如果显示如上所示，表明 Rust 已经安装成功!恭喜你！

这个安装程序还会在本地安装文档的一个副本,这样你就可以离线阅读。在 UNIX 系统中, `/usr/local/share/doc/rust` 是安装的位置。在 Windows 上，它在一个 `share/doc` 目录里，无论你安装 Rust 到哪里。

如果不是如上所示,有很多地方你可以得到帮助。最简单的是在 `irc.mozilla.org` 上的 `# Rust IRC` 频道，你可以通过[Mibbit](http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust)访问它。单击该超链接，你就可以与其他 Rustaceans (我们用来称呼自己的萌萌的昵称)聊天了，我们可以帮助你。其它有用的资源包括[用户论坛](http://users.rust-lang.org/)和 [Stack Overflow](http://stackoverflow.com/questions/tagged/rust)。