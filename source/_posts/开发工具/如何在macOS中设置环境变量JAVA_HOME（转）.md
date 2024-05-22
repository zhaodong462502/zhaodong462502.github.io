## 如何在macOS中设置环境变量JAVA\_HOME

这篇文章将告诉你如何分别在旧的mac osx上和最新的macOS11+上设置JAVA\_HOME

1.  macOS 发布历史
2.  什么是`/usr/libexec/java_home`
3.  $JAVA\_HOME 和 macOS 11 Big Sur
4.  $JAVA\_HOME 和 Mac OS X 10.5 Leopard
5.  $JAVA\_HOME 和 older Mac OS X
6.  在不同的 JDK 版本间切换

**解决办法**  
在macOS上设置 `$JAVA_HOME` 的步骤

1.  明确系统版本
2.  明确你使用的shell类型, bash or zsh?
3.  对于 zsh shell, export `$JAVA_HOME` at `~/.zshenv` or `~/.zshrc`
4.  对于 bash shell, export `$JAVA_HOME` at `~/.bash_profile` or `~/.bashrc`
5.  打印 `echo $JAVA_HOME`
6.  Done。

## 1、macOS发布历史，bash 还是 zsh？

1.1 重温macOS发布历史, source [Wikipedia – macOS](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FMacOS)

1.  Mac OS X Public Beta
2.  Mac OS X 10.0 (Cheetah)
3.  Mac OS X 10.1 (Puma)
4.  Mac OS X 10.2 Jaguar
5.  Mac OS X 10.3 Panther
6.  Mac OS X 10.4 Tiger
7.  Mac OS X 10.5 Leopard
8.  Mac OS X 10.6 Snow Leopard
9.  Mac OS X 10.7 Lion
10.  OS X 10.8 Mountain Lion
11.  OS X 10.9 Mavericks
12.  OS X 10.10 Yosemite
13.  OS X 10.11 El Capitan
14.  macOS 10.12 Sierra
15.  macOS 10.13 High Sierra
16.  macOS 10.14 Mojave
17.  macOS 10.15 Catalina (zsh)
18.  macOS 11 Big Sur (zsh)

1.2 **bash or zsh?**

自 macOS 10.15 Catalina 及之后, 默认的 Terminal shell 从 [bash](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FBash_%28Unix_shell%29) (Bourne-again shell) 换成了 [zsh](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FZ_shell) (Z shell).

-   对于 bash shell, 我们可以设置环境变量的文件是 `~/.bash_profile` 或 `~/.bashrc`
-   对于 zsh shell, 我们可以设置环境变量的文件是 `~/.zshenv` 或 `~/.zshrc`

可以通过环境变量 `$SHELL` 查看你当前使用的shell类型

## 2\. 什么是 /usr/libexec/java\_home

2.1 从Mac OS X 10.5+, 我们可以使用 `/usr/libexec/java_home` 返回默认的jdk路径

2.2 也可以找到所有已安装JDKs

```shell
% /usr/libexec/java_home -V Matching Java Virtual Machines (4): 16 (x86_64) "Oracle Corporation" - "OpenJDK 16-ea" /Library/Java/JavaVirtualMachines/jdk-16.jdk/Contents/Home 15.0.1 (x86_64) "UNDEFINED" - "OpenJDK 15.0.1" /usr/local/Cellar/openjdk/15.0.1/libexec/openjdk.jdk/Contents/Home 14.0.2 (x86_64) "AdoptOpenJDK" - "AdoptOpenJDK 14" /Library/Java/JavaVirtualMachines/adoptopenjdk-14.jdk/Contents/Home 1.8.0_275 (x86_64) "UNDEFINED" - "OpenJDK 8" /usr/local/Cellar/openjdk@8/1.8.0+275/libexec/openjdk.jdk/Contents/Home /Library/Java/JavaVirtualMachines/jdk-16.jdk/Contents/Home
```

2.3 也可以运行特殊的jdk命令

```shell
% /usr/libexec/java_home -v1.8 /usr/local/Cellar/openjdk@8/1.8.0+275/libexec/openjdk.jdk/Contents/Home
```

## 3\. $JAVA\_HOME 和 macOS 11 Big Sur

从 macOS 10.15+, `zsh` 成为默认的 Terminal shell, 我们可以通过 `~/.zshenv` or `~/.zshrc` 设置 `$JAVA_HOME`

~/.zshenv

```bash
export JAVA_HOME=$(/usr/libexec/java_home)
```

## 4\. $JAVA\_HOME 和 Mac OS X 10.5 Leopard

对于 更旧的 Mac OS X版本, `bash` 作为默认的 Terminal shell, 我们可以通过 `~/.bash_profile` or `~/.bashrc` 设置`$JAVA_HOME`

~/.bash\_profile

```shell
export JAVA_HOME=$(/usr/libexec/java_home)
```

## 5\. $JAVA\_HOME 和 更旧的 Mac OS X

对于更旧的 Mac OS X 版本, 则没有 `/usr/libexec/java_home` 这样的工具，我们需要将真实java路径设置到 `$JAVA_HOME` ，如

~/.bash\_profile

```bash
export JAVA_HOME=/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
```

再

## 6\. JDK 版本间切换

比如，当前 macOS 包含 4个 JDK: 1.8, 14, 15, and 16, 和 默认 JDK 16。

```bash
% /usr/libexec/java_home -V Matching Java Virtual Machines (4): 16 (x86_64) "Oracle Corporation" - "OpenJDK 16-ea" /Library/Java/JavaVirtualMachines/jdk-16.jdk/Contents/Home 15.0.1 (x86_64) "UNDEFINED" - "OpenJDK 15.0.1" /usr/local/Cellar/openjdk/15.0.1/libexec/openjdk.jdk/Contents/Home 14.0.2 (x86_64) "AdoptOpenJDK" - "AdoptOpenJDK 14" /Library/Java/JavaVirtualMachines/adoptopenjdk-14.jdk/Contents/Home 1.8.0_275 (x86_64) "UNDEFINED" - "OpenJDK 8" /usr/local/Cellar/openjdk@8/1.8.0+275/libexec/openjdk.jdk/Contents/Home /Library/Java/JavaVirtualMachines/jdk-16.jdk/Contents/Home
```

6.1 根据shell类型打开对应的配置文件如 `~/.zshenv`  
6.2 然后通过`/usr/libexec/java_home -v"{$Version}"` 激活特定的 JDK 版本

如JDK8

~/.zshenv

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v1.8)
```

假如你需要 JDK 14

~/.zshenv

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v14)
```

假如你需要 JDK 15

~/.zshenv

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v15)
```

6.3 Source 文件 并打印 `$JAVA_HOME`, 完成。

```bash
% source ~/.zshenv % echo $JAVA_HOME /usr/local/Cellar/openjdk@8/1.8.0+275/libexec/openjdk.jdk/Contents/Home
```
