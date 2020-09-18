---
title: no matches found
author: Uphie
date: 2022-05-01 13:10:00 +0800
categories: [技术]
tags: [zsh]
math: true
toc: true
---

笔者在使用 touch 命令时，提示如下错误，原始命令是 `touch some?.md`
```
zsh: no matches found: some?.md 
```
操作系统是mac OS，shell是 zsh。

看样子，shell在处理文件名时，将文件名中的 `?` 当成了匹配字符并对文件进行了查找，而我的预期是将 `?` 当成普通字符就好。


解决方法：

编辑 `~/.zshrc`，增加以下内容：
```
setopt no_nomatch
```
最后 `source ~/.zshrc` 使之生效。

再重试，OK。

参考资料：

[https://zsh.sourceforge.io/Doc/Release/Options.html#Options](https://zsh.sourceforge.io/Doc/Release/Options.html#Options)