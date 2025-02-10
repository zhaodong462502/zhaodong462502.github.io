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

## 五、Git Lfs

### 背景

git lfs 是git的一个插件工具，可以实现大文件的提交。例如在github仓库中，默认提交的文件大小不大于100M。大文件可以使用git lfs命令。

> AI大模型的文件普遍都很大，所以都是采用git lfs方式提交仓库。

### 使用方法

#### 1. 下载git fls

##### mac

```cmd
//mac
brew install git-lfs

```

##### windows

访问 [Git LFS 官方发布页面](https://github.com/git-lfs/git-lfs/releases)，下载适用于 Windows 的最新版本的安装程序（通常是 `.exe` 文件）。

运行下载的安装程序，按照安装向导的提示完成安装。

安装完成后，打开命令提示符或 PowerShell，输入 `git lfs install` 来初始化 Git LFS，该命令会在全局配置中启用 Git LFS。

#### 2. 跟踪大文件

在 Git 仓库中，你需要指定哪些类型的文件或具体哪些文件要使用 Git LFS 进行管理。

- **跟踪特定类型的文件** 若要跟踪所有 `.psd`（Adobe Photoshop 文件）、`.mp4`（视频文件）等特定扩展名的文件，可使用 `git lfs track` 命令，示例如下：

```bash
git lfs track "*.psd"
git lfs track "*.mp4"
```

此命令会在仓库根目录创建或更新一个名为 `.gitattributes` 的文件，其中会记录跟踪的文件类型，比如：

```plaintext
*.psd filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
```



- **跟踪特定文件** 若只想跟踪某个具体的大文件，例如 `large_file.zip`，可使用以下命令：

```bash
git lfs track "large_file.zip"
```



#### 3. 添加和提交文件

在跟踪文件之后，就可以像使用普通 Git 操作一样添加和提交文件。

```bash
git add .gitattributes  # 添加 .gitattributes 文件到暂存区
git add large_file.zip  # 添加跟踪的大文件到暂存区
git commit -m "Add large file using Git LFS"  # 提交更改
```



#### 4. 推送文件到远程仓库

使用 `git push` 命令将本地仓库的更改推送到远程仓库。

```bash
git push origin main  # 假设推送到 main 分支
```

当执行 `git push` 时，Git LFS 会将大文件上传到其对应的存储服务（如 GitHub 提供的存储服务），而在 Git 仓库中仅存储文件的指针。

#### 5. 克隆包含 Git LFS 文件的仓库

当你克隆一个使用了 Git LFS 的仓库时，需要确保获取大文件的实际内容，而不仅仅是指针。



- 使用 `git lfs clone` 命令直接克隆：

```bash
git lfs clone <repository_url>
```



- 若已经使用普通的 `git clone` 克隆了仓库，可以在仓库目录下运行以下命令来获取大文件的实际内容：

```bash
git lfs pull
```



#### 6. 查看跟踪的文件



若想查看当前仓库中使用 Git LFS 跟踪的文件列表，可使用以下命令：

```bash
git lfs track
```

#### 7. 停止跟踪文件

若不再需要使用 Git LFS 跟踪某个文件或某类文件，可按以下步骤操作：

- 编辑 `.gitattributes` 文件，删除对应的跟踪规则。
- 然后执行以下命令将文件从 Git LFS 管理中移除：

```bash
git lfs untrack "*.psd"  # 停止跟踪 .psd 文件
git add .gitattributes
git commit -m "Stop tracking .psd files with Git LFS"
```

不过要注意，已经使用 Git LFS 上传的文件记录仍然会保留在存储服务中。
