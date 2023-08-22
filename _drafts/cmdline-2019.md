---
title: "My Command Line Solution Roundup - 2019"
date: 2020-01-24T20:07:37+08:00
draft: true
tags: []
categories: []

toc: true
mathjax: false
---

在这个时间节点上，还是稍微来整理下过去这一年以来在命令行工具方面的积累吧。过去的一年也是我在命令行中工作地越来越多的一年——中学的时候写了不少东西，学了不少操作，但是现在看来，其实都非常地扁平；随着项目规模的扩大，还是需要更好地工具来武装自己，提高效率。

另外一点就是，有时候我们会觉得在图形环境下很容易工作，但在命令行环境下感到困难。所以我觉得一个比较有帮助的事情是，来整理一下我们如何使用一些图形的、集成的工具，从需求的角度出发，来整理命令行的使用技巧。

Here we go!

## Why Windows

往前两年特别痴迷于Linux环境，那时候觉得`nautilus`和`gnome-terminal`都是工作必备——我至今觉得`nautilus`是世界上最好的文件管理器，Linux上的Terminal选择范围也要大很多。但很遗憾，我的工作全部并不只是文件管理器和命令行，我还要上课，还要修图，还要摸鱼。所以，借着换新电脑的契机，我重新开始将Windows作为主要平台。

[Git for Windows](https://gitforwindows.org/)是个非常好的工具，提供了SSH和Git这种基本工具，还提供了个Bash工具箱；当然，没啥扩展性。微软近年来也推出了WSL，不过细想还是有很多坑，比如WSL的API不健全，桥接的文件系统，糟糕的IO性能，以及操作上其他GUI工具如何使用WSL中的工具？尤其是如果涉及到SSH和Git，其中其实是有一些坑点的；这些都并非不能解决的问题，但都是需要解决的问题，所以我还是在用着Git Bash，用着Native的Python。但WSL也并非没有用处，我依赖WSL来作为Git Bash的补充，在里面查看manpage、安装编译器等工具。两者互为补充，在大部分时候已经可以替代Linux了，我还放了一个Xubuntu的虚拟机作为备用。

关于文件系统，NTFS上当然不存在权限设置，但是实际使用中这并不是一件特别糟糕的事情，Git for Windows默认设置了权限不敏感，如果要设置一个文件的权限可以用`update-index`，而且大部分时候其实都是`clone`现有的repo，所以对Git来说没啥影响；反而，不要用WSL里的Git。软链的话，NTFS其实是支持的，可以在cmd里用`mklink`指令，而且Win10现在打开开发者模式就可以不需要管理员权限了。真正比较大的影响其实是路径大小写不敏感，放到NTFS上可能会出现路径相互覆盖。

总之，NTFS是个麻烦，但现在都能绕开了，应该说赶上了MSYS成熟、WSL启动的好时代；Linux上有的工具确实是神一般的好用，但代价是另外有的工具几乎就不可用。mac么，早年有人说他兼具Windows与Linux之长，但实际上也是兼具两者之短，键盘和接口上的遗憾也是很大的问题。

## Why Git Bash

关于Terminal的选择，我还是尝试了不少选择，最终最喜欢的还是Git for Windows自带的Git Bash (MinTTY)。简单，快速，功能齐全，除了没有标签页，没有什么好抱怨的。[MobaXterm](https://mobaxterm.mobatek.net/)也是个好东西，自带X Server，让我偶尔连集群图形不再需要Linux虚拟机或者x2go，但我对于启动速度并不怎么满意，其配置的眼花缭乱也令我敬而远之。对ConEmu的远离大抵也是因此，更何况我都没有把鼠标的问题解决出来。我也尝试过[hyper](https://hyper.is/)之类基于Electron的方案，在里面跑Git Bash的Shell，在解决了一些问题之后也还不错，但并不明显由于MinTTY吧。Windows Terminal初听觉得石破天惊，但是到现在仍然处于beta，也就不再提了。

值得一提的是，我很喜欢[Termius](https://www.termius.com/)，跨平台，可以同步（虽然我没有），而且还挺好看。不过在Windows客户端也没有标签页，虽然多窗口是有的，这本质上是个SSH客户端，但也可以执行本机shell，Git Bash是可以的，各种功能支持也还算健全。

前面说过，Git Bash是一个很不错的工具箱。SSH命令行，能够支持端口转发之类的功能，能够自己控制自己的密钥；sed、awk等脚本神器；一个简单的vim。我一度通过Git Bash来做一些Windows管理的工作，比如打包配置文件，还写了一些成规模的Shell脚本，但当然，我现在还是都回归了Native Python来完成这些工作——更好的可维护性，更好的性能，还是很香的。

更重要的一点是，Git Bash其实是个非常不错的Launcher。自带`winpty`，可以执行powershell和cmd，也可以安装[wslbridge](https://github.com/rprichard/wslbridge)，这个工具支持了广受赞誉的wsltty。

## 我的Git Bash配置

我基本上是香草（vanilla）派，外观上没有做太多的定制，从功能上倒是做了些修改。Git Bash主要还是用来连SSH，所以定制也还是从这方面着手。

开始入门SSH的时候基本上看的是GitHub的[手册](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)，从生成密钥开始，后来又觉得明文密钥不安全，于是加上了口令。然而加入口令后麻烦其实真的不少：IDE里`git`不能用了，每次命令行连接也都要输密码，不过好歹每台服务器输的是同样的密码。

Git Bash没有标签页也算是个问题。我也试过基于Electron的方案，比如VSCode，Termius之类的，接着就发现中文也不支持了，`alias`也不全了。检查了一下，发现是login shell的问题。于是我开始在`.bashrc`里`source`一些基本的profile。

在这个过程中，发现了`start-ssh-agent.cmd`这么个脚本，自此开始折腾agent。想法很简单，开机启动agent，然后之后的`ssh`就都可以不用重复输密码了。但是Windows上不存在Login Shell，那环境变量该存在哪里呢？我试着用`setx`设Windows环境变量，效果很好，`git`也能够在VSCode里用了。但使用中还是有问题，经常会有智障清理软件把Socket File给删掉，这样的话agent就必须杀掉重新起；在VSCode里起起来的Git Bash会从Code继承旧的环境变量，结果还是会连不上agent。于是最近更新了一次，每次启动Git Bash都尝试启动agent，而且把环境存在一个文件中，这样每个子shell可以自己加载环境，不受继承的影响。

另外我还做了一个基于`.ssh/config`的SSH Host补全。上述的这些功能我都放在[我的GitHub](https://github.com/Tantalus13A98B5F/dotfiles/blob/master/bashrc.gitbash)上了。

## 我的Shell配置

早年还是用过[Oh My Zsh](https://ohmyz.sh/)的，一开始觉得这东西好用又好看，然而回到Windows上，这就成了大坑。Git Bash里没带ZSH，那就用WSL吧；结果放在WSL上，严重拖慢启动速度。我最近回头看ZSH的配置，发现很多人其实都不推崇OMZ这个配置，认为Git Branch显示就是个坑。当然而且，Windows上的各个Terminal和其中的字体对于OMZ里用到的字符支持都不算好，这加速了我的退坑。

以使用服务器为契机，我还是主要使用Bash了。作为香草派，我的`PS1`设置还是很简单的：以Debian的设置为基础，把全路径名换成了路径末端名。这样的话路径显示不会太长，而且不同的命令之间色彩上能看出来隔离。

我对ZSH的特性利用还是比较局限，用的最多的就是上翻历史。当然，知道了Bash里的`^r`快捷键之后，我觉得Bash的体验也可以接受了。

在调CESM的时候意识到，命令行用起来烦的一个很重要的原因是在不同的路径间切换很麻烦。ZSH在这方面的支持其实比较不错，但我当时因为没有需求并没有去了解。当然，有一些外部的工具，如`autojump`之类的，可以统计频率，局部匹配，自动跳转；但速度也是个问题，而且毕竟不原生。我另外用过的一个办法是`mc`——Midnight Commander这个文件管理器可以两个路径并排，并分别进入对应路径的Shell。好用是好用，但只能支持两个路径嘛，而且我有一次犯了糊涂进了嵌套`mc`，结果相当惨烈。

也许这个需求可以用`tmux`来解决？然而工作区并不一定能够继续划分。我最近用得比较多的是Bash里的`pushd`功能，相当于是一个简易的标签页——这就和图形的操作比较像了。虽然手动了一点，但是方便和效率上都还是不输的。

解决这个问题的另一种思路是模糊匹配工具——我在Vim里装上了`fzf`工具，就像IDE一样，也就不再需要在路径树里翻来翻去找文件了。

## 我的tmux配置

一开始我觉得我对`tmux`的需求很简单，能用鼠标就行了。然而不同版本设置鼠标的命令还是不一样的。后来使用中又发现的一个问题：有的版本的`tmux`中，fork出来的pane会继承路径，有的则会又从头开始。这时候我开始感到迷惑——这个工具是真的不讲究后向兼容性啊，而且怎么还有新版本砍功能的情况。

于是我开始去查能不能写多版本的配置文件，还真找到了。我现在用的配置文件是通过`awk`来解析和比较版本的，这种方式看网上用得不多，可能`awk`的兼容性会有问题？但可读性相对来说是非常好的，而且在我使用的几个平台上表现良好。

最近发现同事很多都用[tony/tmux-config](https://github.com/tony/tmux-config)，于是也去试了一下，发现Windows上糟糕的Terminal色彩还是支撑不起这样的配置——所以默认配置是有默认配置的道理的。不过我还是去合并了一些功能性的配置，近来试用了一下，感觉舒服了不少。虽然都是些老生常谈的东西，但是还是再提一下吧：

- 用`C-a`作为prefix。虽然终极方案是把Ctrl和CapLocks换掉但是……先就这样吧。不过比较有趣的是，这个配置还设置了两下`C-a`切换窗口，活脱脱一个盗版`screen`。虽然很多人主张`C-a`用得比`C-b`少，但`C-a`可是经典emacs键位跳转到行首，直接封掉还是很可惜的；`C-z`、`C-s`之流就更令我感到困惑了。所以我还是用`C-a`，但是加了个`send-prefix`。
- 窗口编号从1开始。
- 窗口之间的切换可以用`hjkl`，比上下左右舒服了不少。
- 关于复制模式，虽然Vi模式应该更加的人类友好，但emacs键位意外地非常还原，出于练习emacs的目的可能会用一段时间；是的，Vi模式就不怎么还原，至少说不支持Vim中的`v`和`y`指令，抄来的配置中也修复了这个情况。

我的tmux配置也放在[GitHub](https://github.com/Tantalus13A98B5F/dotfiles/blob/master/tmux.conf)上了。

## 我的编辑器选择

随着在服务器上的工作越来越多、自己的虚拟机也不怎么撑得起VSCode的内存消耗，逐渐感到在服务器上高效编辑还是很重要的一门技能，而不是临时用用怎么样都行。当然，VSCode现在支持了远程编辑，这非常厉害，但另一方面是我发现，在tag系统的加持下，Vim的可用性一点不比Code差。

我也从我的同事那里要来了他们的Vim配置，不过还没来得及细看。作为有一定练度的用户了，也已经不像初学者一样拿上一个配置就可以直接用，阅读与思考还是很重要。尤其是对于编辑器来说，这就是开发者工作的最重要的环境了，考虑清楚当中的需求还是非常重要的。

关于Vim和Emacs之争，我是Vim老用户了，Emacs则是学了几次都没学会——有一说一，我觉得Emacs上手教程把退出放在最后讲是个败笔，一时半会没看完又连退出都不会，几次都气急败坏直接杀掉进程干别的事去了。但Emacs还是很重要，至少这套键位——虽然经常被认为是程序员工伤的罪魁祸首之一——使用的范围非常广泛。哪怕有人在Emacs里用Vim键位，在我对Emacs本身进一步产生兴趣之前，这都与我无关了。（有一说一Emacs调试好像厉害很多，有机会要试试）

我在开始搭建我的Vim配置之前罗列了一下我现在觉得很需要的功能，Vim自带的有以下一些：

- 多标签、文件并排：众所周知(?) Vim是有Buffer功能的，但是开过的所有窗口都会进入Buffer，标签页在我看来则是一种显式的固定，应当是正交的。Vim有这方面的功能，但快捷键可能需要再配置下；
- 自动补全：因为目前看得多写得少，基本上自带的文本补全就够了；
- 代码的折叠、缩进、注释：然而还没有试用过相关的功能；

从一些现代编辑器中感到必要性的功能包括：

- 文件模糊匹配：我用了[`fzf.vim`](https://github.com/junegunn/fzf.vim)，除了文件名还能搜很多别的东西，用处巨大；
- 多处编辑：[`vim-multiple-cursors`](https://github.com/terryma/vim-multiple-cursors)基本够用，而且不需要怎么配置；
- 代码跳转：也是Vim中自带的Tag系统，但还需要一些外部工具的配合，接下来展开说；

要实现代码跳转，需要编辑器理解程序的语言吗？其实不怎么需要。有一类程序被称为`*tags`，则是通过简单的词法分析，从程序中提取定义的符号。这里面最经典的要数`ctags` (exuberant)，在项目目录里运行`ctags -R .`后会生成一个名为`tags`的文件，包含该项目中所有的符号和对应的位置。Vim能够读取`tag`文件中的信息，在有tag信息的目录下打开文件后，将光标移到一个引用的符号下，按下`^]`，就能够跳转到符号定义的位置。当然还有些别的功能，例如返回跳转前的位置、在新的标签或窗口中打开跳转。总之，这种看起来高级的功能，Vim其实是支持的，而且支持的语言相当多，只要`ctags`支持的都可以。

当然这种外部程序的方法是会有一些痛点的，比如编辑之后需要手动重新索引，不支持增量更新的话重新索引一次可能会需要非常长的时间。还有一个问题是，`ctags`只能搜索定义，不能搜索引用。因此要用到`cscope`。这个程序使用逻辑和`ctags`差不多，也需要先新建索引，但其本身就是一个可以独立使用的交互式程序，可以在几种不同的类别下搜索符号，搜索的结果可以用Vim打开。当然，这个程序基本上就只支持C语言，而且每个结果的查看都在不同的的Vim窗口下。

```text
Find this C symbol:
Find this function definition:
Find functions called by this function:
Find functions calling this function:
Find this text string:
Change this text string:
Find this egrep pattern:
Find this file:
Find files #including this file:
Find assignments to this symbol:
```

`cscope`在Vim中使用上也要麻烦一些。需要先用`:cs add <file>`添加数据库，然后用`:cs find <mode> <sym>`进行搜索。虽然比`ctags`功能强，但好像并不能替代`ctags`；其实也有办法，可以设置`:set cscopetag`，这样定义跳转功能也可以不经`ctags`就使用了。

另外一个值得提到的工具是`gtags`，或者叫`GNU Global`。`gtags`可以直接在目录里生成可增量更新的数据库文件，共计3个文件。相对来说比`ctags`是复杂了一些，但是`gtags`可以生成对引用的索引。`gtags`附带了一个`global`程序，可以用来对数据库进行检索——毕竟不同于`ctags`的索引文件，那个是文本形式的。为了在Vim中使用，`gtags`自带了一个`Gtags.vim`插件，不过并不支持前述的那套`ctags`的操作。

总体来说`gtags`提供了比`ctags`和`cscope`都要好的性能，比`cscope`更好的语言兼容性，所能提供的功能至少和`cscope`相同，但是没有特别好的Vim接口。但是正因为`gtags`能够完成`cscope`的主要功能，`gtags`程序包也提供了一个`gtags-cscope`命令，通过`cscope`的操作方式和接口，以`gtags`为后端来完成检索。是的，可以用`gtags-cscope`来替代`cscope`，在Vim中设定`:set cscopeprg=gtags-cscope`即可。

查找定义和查找引用在代码阅读中就算是比较典型的场景了，`ctags`和Vim的契合度是最好的，但是不支持查找引用；`gtags`则提供了更好的功能，通过`cscope`接口稍加配置使用体验也很不错，总比`grep`搜索要好。当然这里面还有些手动的过程，比如Tagfile的更新，`cs add`的调用，不过应该已经有相应的插件了。