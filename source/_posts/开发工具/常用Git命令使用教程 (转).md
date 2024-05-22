## 0x00 写在前面

现在大部分的开发团队都以 Git 作为自己的版本控制工具，需要对 Git 的使用非常的熟悉。这篇文章中本人整理了自己在开发过程中经常使用到的 Git 命令，方便在偶尔忘记时速查。使用 GUI 工具的同学，也可以对照起来看看。

![常用 Git 命令使用教程](https://segmentfault.com/img/remote/1460000011673668?w=948&h=1010 "常用 Git 命令使用教程")

## 0x01 Git 配置

1\. 在安装完成 Git 后，开始正式使用前，是需要有一些全局设置的，如用户名、邮箱。

设置的主要命令是 `git config`:

```stylus
git config --global user.name "your name"      
git config --global user.email "your email"    
```

2\. 除了用户名、邮箱之外，还有很多的配置可以用来自定义 Git，如：

```arduino
git config --global color.ui true        
git config core.ignorecase true            
```

```arduino
git config -l
```

## 0x02 基础操作

### 1\. 创建 Git 版本库

在本地创建 Git 版本库，需要使用 `git init` 命令。

首先，你需要新建一个存放版本库的目录，然后进入到该目录所在路径，然后执行：

```csharp
git init
```

![git init](https://segmentfault.com/img/remote/1460000011673670?w=647&h=459 "git init")

### 2\. 将文件添加到版本库

要将一个文件纳入到版本库管理，首先要将其添加到暂存区(这里不做深入介绍)，然后才能提交到仓库中。

2.1 将文件添加到暂存区，使用的是 `git add`：

```armasm
git add Readme.md        
git add .                
```

![git add](https://segmentfault.com/img/remote/1460000011673671?w=647&h=459 "git add")

2.2 将暂存区中的文件，提交到仓库中。需要使用 `git commit`：

```awk
git commit        // 如果暂存区有文件，则将其中的文件提交到仓库
git commit -m 'your comments'         // 带评论提交，用于说明提交内容、变更、作用等
```

![git commit](https://segmentfault.com/img/remote/1460000011673672 "git commit")

### 3\. 查看仓库的状态

不论我们是新建了文件，将文件加入暂存区，或者其他的修改等等，我们都可以通过：

```ebnf
git status
```

### 4\. 查看仓库中的具体修改

很经常的，我们对某个文件做了修改，但过不久就忘记了。这时候就可以通过 `git diff` 来查看具体的修改内容。

```awk
git diff    // 查看版本库中所有的改动
git diff Readme.md        // 查看具体文件的改动
```

### 5\. 查看提交历史记录

有的时候，你会需要查看自己做过哪些提交，来回顾自己完成的部分。或者需要寻找某个具体的提交来查看当时的代码。这里需要用到：

```1c
git log     
git log --pretty=oneline    
```

![git log](https://segmentfault.com/img/remote/1460000011673674?w=647&h=459 "git log")

### 6\. 版本回退

有了 `git log` 来查看提交的历史记录，我们就可以通过 `git reset --hard` 来回退到我们需要的特定版本，然后使用当时的代码进行各种操作。

```awk
git reset --hard HEAD^        // 回退到上一个提交版本
git reset --hard HEAD^^        // 回退到上上一个提交版本
git reset --hard 'commit_id'    // 会退到 commit_id 指定的提交版本
```

### 7\. 回到未来的某个提交

当退回到某个提交的版本以后，再通过 `git log` 是无法显示在这之后的提交信息的。但是，通过 `git reflog` 可以获取到操作命令的历史。

因此，想要回到未来的某个提交，先通过 `git reflog` 从历史命令中找到想要回到的提交版本的 ID，然后通过 `git reset --hard` 来切换。

```dsconfig
git reflog
git reset --hard 'commit_id'
```

### 8\. 撤销修改

撤销修改同样包括两方面的内容，由于仓库中的文件在提交之前，可能在工作区中，尚未在版本控制范围内，也可能在暂存区中。

8.1 丢弃工作区中文件的修改

```awk
git checkout -- Readme.md    // 如果 Readme.md 文件在工作区，则丢弃其修改
git checkout -- .            // 丢弃当前目录下所有工作区中文件的修改
```

![git checkout --](https://segmentfault.com/img/remote/1460000011673677?w=647&h=459 "git checkout --")

8.2 丢弃已经进入暂存区的修改

```awk
git reset HEAD Readme.md     // 将 Readme.md 恢复到 HEAD 提交版本的状态
```

### 9\. 删除文件

在文件未添加到暂存区之前，对想删除文件可以直接物理删除。或者通过 `git checkout -- file` 来丢弃。如果文件已经被提交，则需要 `git rm` 来删除：

```awk
git rm Readme.md     // 删除已经被提交过的 Readme.md
```

![git rm](https://segmentfault.com/img/remote/1460000011673679?w=647&h=459 "git rm")

**`git rm` 与 先 rm 然后 `git add` 的区别**

![rm and git add](https://segmentfault.com/img/remote/1460000011673680?w=647&h=459 "rm and git add")

更详细的可以参考：["git rm" 和 "rm" 的区别](https://link.segmentfault.com/?enc=DzE%2FT6RbWpi21ObzWKgm2g%3D%3D.ty4wWrTMJ5SDwEFkkERce9x%2FkcwA9caEUV3C61MTJgsqaOSF4jTzXmzXx5rlTFtIWkmsmMVtDLx4MBAOSEU64KtuDSIRAFGC6gt9DH6mZQQ%3D)

> 注意：上图中的结果是在 git 1.9.1 版本上的操作。在 git 2.0 以上两者没有区别了。

## 0x03 分支管理

分支是版本控制系统中很重要的一个概念，在 Git 中新建、合并等分支的操作非常轻量便捷，因此我们会很经常的用到。

### 1\. 查看分支

查看分支使用 `git branch`：

```awk
git branch        // 查看本地分支信息
git branch -v     // 查看相对详细的本地分支信息
git branch -av     // 查看包括远程仓库在内的分支信息
```

### 2\. 创建分支

当我们要修复一个 Bug，或者开发一个新特性，甚至是在初学的时候怕打乱原来的代码，都可以新建一个分支来避免对原来代码的影响。

```awk
git branch dev    // 新建一个名称为 dev 的分支
```

当我们创建完分支以后，我们需要切换到新建的分支，否则，所有的修改，还是在原来的分支上。事实上，所有的改动，只能影响到当前所在的分支。

```awk
git checkout dev    // 新建完 dev 分支以后，通过该命令切换到 dev 分支
```

```armasm
git checkout -b dev        
```

### 5\. 合并分支

当我们修复完成一个 Bug，或者开发完成一个新特性，我们就会把相关的 Bug 或者 特性的上修改合并回原来的主分支上，这时候就需要 `git merge` 来做分支的合并。

首先需要切换回最终要合并到的分支，如 `master`：

```crmsh
git checkout master        // 切换回 master 分支
git merge dev            // 将 dev 分钟中的修改合并回 master 分支
```

### 6\. 删除分支

当之前创建的分支，完成了它的使命，如 Bug 修复完，分支合并以后，这个分支就不在需要了，就可以删除它。

```awk
git branch -d dev        // 删除 dev 分支
```

上面的所有命令都是针对本地仓库的操作。当我们希望多个人来协作时，会将代码发布到一个统一的远程仓库，然后多个人在本地操作以后，在推送到远程仓库。其他人协作时，需要先同步远程仓库的内容，再推送自己的修改。

### 1\. 从远程仓库克隆

如果你本地没有仓库，希望从已有的远程仓库上复制一份代码，那么你需要 `git clone`。

```awk
git clone https://github.com/git/git.git     // 通过 https 协议，克隆 Github 上 git 仓库的源码
git clone linfuyan@github.com/git/git.git    // 通过 ssh 协议，克隆 Github 上 git 仓库的源码
```

### 2\. 添加远程仓库

如果你已经有了一个本地仓库，如之前创建的 `git-guide`，然后你打算将它发布到远程，供其他人协作。那么使用：

```armasm
git remote add origin your_remote_git_repo        
```

当本地仓库中，代码完成提交，就需要将代码等推送到远程仓库，这样其他协作人员可以从远程仓库同步内容。

```crmsh
git push -u origin master // 第一次推送时使用，可以简化后面的推送或者拉取命令使用
git push origin master    // 将本地 master 分支推送到 origin 远程分支
```

### 4\. 从远程仓库获取最新内容

在多人协作过程中，当自己完成了本地仓库中的提交，想要向远程仓库推送前，需要先获取到远程仓库的最新内容。

可以通过 `git fetch` 和 `git pull` 来获取远程仓库的内容。

```crmsh
git fetch origin master    
git pull origin master
```

-   `git fetch` 是仅仅获取远程仓库的更新内容，并不会自动做合并。
-   `git pull` 在获取远程仓库的内容后，会自动做合并，可以看成 `git fetch` 之后 `git merge`。

> 注意：建议多使用 `git fetch`。

### 5\. 查看远程仓库信息

```awk
git remote [-v]        // 显示远程仓库信息
```

在本地仓库中的分支和远程仓库中的分支是对应的。一般情况下，远程仓库中的分支名称和本地仓库中的分支名称是一致的。

有的时候，我们会需要指定本地分支与远程分支的关联。

```actionscript
git branch --set-upstream 'local_branch' origin/remote_branch
```

当远程的仓库地址发生变化时，需要修改本地仓库对应的远程仓库的地址。主要应用在[工程迁移](https://link.segmentfault.com/?enc=a9TSZRBvDgD%2F9iTimaINWw%3D%3D.LhMLXyxg19JDkcnGUFTIke7NJy719SUyB5EUcD8E%2B2jUT%2Fh4Oz%2BCMG%2BKS1sOIkHpnSbqi1b7I%2Fd7zBKEWa2Row%3D%3D)过程中。

```dsconfig
git remote set-url origin url
```

在项目开发过程中，当一个版本发布完成时，是需要对代码打上标签，便于后续检索。获取处于其他的原因，需要对某个提交打上特定的标签。

### 1\. 创建标签

```lasso
git tag -a 'tagname' -m 'comment' 'commit_id'
```

### 2\. 查看所有标签

```awk
git tag        // 查看本地仓库中的所有标签
```

```dart
git show tagname
```

如果打的标签出错，或者不在需要某个标签，则可以删除它。

```crmsh
git tag -d tagname
```

```maxima
git push origin :refs/tags/tagname

git push origin --delete tagname

git push origin :tagname
```

打完标签以后，有需要推送到远程仓库。

6.1 推送单个标签到远程仓库

```maxima
git push origin tagname
```

```maxima
git push origin --tags
```

### 1\. 临时保存修改

在执行很多的 Git 操作的时候，是需要保持当前操作的仓库/分支处于 clean 状态，及没有未提交的修改。如 `git pull`， `git merge` 等等，如果有未提交的修改，这些将无法操作。

但是做这些事情的时候，你可能修改了比较多的代码，却又不想丢弃它。那么，你需要把这些修改临时保存起来，这就需要用到 `git stash`。

1.1 **临时保存修改**，这样仓库就可以回到 clean 状态。

```awk
git  stash        // 保存本地仓库中的临时修改。
```

1.2 **查看临时保存**。当你临时保存以后，后面还是要取回来的，那它们在哪里呢？

```awk
git stash list    // 显示所有临时修改
```

```awk
git stash apply        // 恢复所有保存的临时修改
git stash pop        // 恢复最近一次保存的临时修改
```

```arduino
git stash clear        
```

这些是我目前在项目中经常会用到的操作，这里整理下来，可以作为一个手册。对于 Git 的理解或者更多的解释，并不在这里体现。大家可以参考其他更多的资料。

文中的思维导图由于从源文件导出成图片之后会变得模糊，关注微信公众号 ID: up2048，回复“脑图”可以免费获取思维导图源文件。
