## 一、.gitIgnore使用

> #隐藏当前目录下的testFile目录及以下的目录文件，不隐藏当前目录下的testFile文件
> /testFile
>
> #隐藏所有目录下testFile目录下的目录文件
> testFile/ 或者 testFile



## 二、sourceTree使用

sourceTree和大多数git客户端（如tower）都不会显示一个空文件夹

如果只是建了一个空文件夹，待提交列表可能识别不到



## 三、Git冲突：Please commit your changes or stash them before you merge

### 问题原因

其他人修改了某个文件并提交到版本库中去了，而你本地也修改了同一个，这时候你进行拉取就会出现冲突了，解决方法，原则是不要去更改别人已经提交的代码，如果确实要更改（不建议也不需要），请先跟当事人沟通

### 使用git stash

```
git stash
git pull
git stash pop
```

>git stash：保存当前工作进度，能够将所有未提交的修改（工作区和暂存区）保存至堆栈中，用于后续恢复当前工作目录。也可以用git stash save，作用等同于git stash，区别是可以加一些注释
>git pull：这个应该不用说了吧！（把服务器仓库的更新拉到本地仓库中）
>git stash pop：可以把你刚才stash到本地栈中的代码pop到本地（也可以用git stash apply，区别：使用apply恢复，stash列表中的信息是会继续保留的，而使用pop恢复，会将stash列表中的信息进行删除。）



## 四、git本地新建仓库并推送远程且远程仓库也是新建

### 结论

> 并不能直接通过命令在远程新建仓库。IDEA中是根据github的api接口创建仓库，然后在和本地仓库进行关联。

以IDEA中发布到Github为例

```
19:36:12.273: [GitSmallFileDemo04] git -c credential.helper= -c core.quotepath=false -c log.showSignature=false init
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint:
hint: 	git config --global init.defaultBranch <name>
Initialized empty Git repository in /Users/zhaodong/dev_project/github_project/GitSmallFileDemo04/.git/
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint: 	git branch -m <name>
19:36:14.861: [GitSmallFileDemo04] git -c credential.helper= -c core.quotepath=false -c log.showSignature=false add --ignore-errors -A -- .idea/material_theme_project_new.xml .idea/vcs.xml .idea/.gitignore .idea/compiler.xml
19:36:14.878: [GitSmallFileDemo04] git -c credential.helper= -c core.quotepath=false -c log.showSignature=false commit -m "Initial commit" --
[master (root-commit) f0767f0] Initial commit
 4 files changed, 32 insertions(+)
 create mode 100644 .idea/.gitignore
 create mode 100644 .idea/compiler.xml
 create mode 100644 .idea/material_theme_project_new.xml
 create mode 100644 .idea/vcs.xml
19:36:14.896: [GitSmallFileDemo04] git -c credential.helper= -c core.quotepath=false -c log.showSignature=false push --progress --porcelain origin master --set-upstream
Enumerating objects: 7, done.
Counting objects:  14% (1/7)
Counting objects:  28% (2/7)
Counting objects:  42% (3/7)
Counting objects:  57% (4/7)
Counting objects:  71% (5/7)
Counting objects:  85% (6/7)
Counting objects: 100% (7/7)
Counting objects: 100% (7/7), done.
Delta compression using up to 16 threads
Compressing objects:  16% (1/6)
Compressing objects:  33% (2/6)
Compressing objects:  50% (3/6)
Compressing objects:  66% (4/6)
Compressing objects:  83% (5/6)
Compressing objects: 100% (6/6)
Compressing objects: 100% (6/6), done.
Writing objects:  14% (1/7)
Writing objects:  28% (2/7)
Writing objects:  42% (3/7)
Writing objects:  57% (4/7)
Writing objects:  71% (5/7)
Writing objects:  85% (6/7)
Writing objects: 100% (7/7)
Writing objects: 100% (7/7), 1008 bytes | 1008.00 KiB/s, done.
Total 7 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To github.com:/zhaodong462502/GitSmallFileDemo04.git
*	refs/heads/master:refs/heads/master	[new branch]
branch 'master' set up to track 'origin/master'.
Done

```

![image-20250208194118676](https://s2.loli.net/2025/02/08/wZ2BeOlkSWac3Iu.png)
