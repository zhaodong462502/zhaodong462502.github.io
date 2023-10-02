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
