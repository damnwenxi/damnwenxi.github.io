+++
title = "你不知道的Git"
date = 2023-11-09 21:54:48
author = "Damnwenxi"
tags = ["Technology", "Git"]
keywords = ["Git"]
cover = "posts/coding/git.png"
readingTime = true
description = "Git 是一个分布式版本控制系统，用于管理软件开发中的源代码和版本控制。它是许多开源项目和商业项目的首选版本控制系统之一。Git 由芬兰程序员Linus Torvalds（李纳斯，Linux之父）开发并于2005年正式发布，由于linux内核开源项目需要，他花了2周时间用c语言写出了git..."
+++

# 你不知道的 Git

---

### 简介&基本概念

Git 是一个 **分布式版本控制系统** ，用于管理软件开发中的源代码和版本控制。它是许多开源项目和商业项目的首选版本控制系统之一。

Git 由芬兰程序员 **Linus Torvalds（李纳斯，Linux 之父）** 开发并于 2005 年正式发布，由于 linux 内核开源项目需要，他花了 2 周时间用 c 语言写出了 git。

#### 分布式

与 “SVN” 不同的是，git 不仅仅在远端存储了代码，同时在每一位用户的本地也有一份拷贝，这也正是 git 保证 **可靠性** 和 **高效率** 的基础。

![repository](/posts/coding/git/repository.png)

#### 存储原理

“源代码&版本控制”

Git 保存的不是文件的变化或者差异，而是一系列不同时刻的 **快照**。

在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个快照流。

#### 三种状态

-   **已修改（modified）**表示修改了文件，但还没保存到数据库中。
-   **已暂存（staged）**表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
-   **已提交（committed）**表示数据已经安全地保存在本地数据库中。

#### 三个阶段

工作区、暂存区、本地目录（仓库）

![areas](/posts/coding/git/areas.png)

#### 分支

为什么需要分支？

使用分支可以让你的工作从开发主线上分离开来，消除 “我的修改会不会影响到原来的代码” 这种顾虑。并且可以让你在多人协同开发过程中互不打扰。git 鼓励开发者频繁的创建和使用分支，因为 git 的分支模型可以保证在管理大量分支的情况下依旧保持高效。

---

### 从 `git clone` 开始，git 到底做了哪些事情？

每当我们从远端关联或者初始化了一个仓库到本地时，都会在项目根目录产生一个 `.git` 文件夹，这就是 git 的版本库，你可以把它理解为一个帮助我们管理代码和版本的数据库。

![image-20231103145039469](/posts/coding/git/git-dir.png)

-   hooks: 客户端和服务端钩子脚本

-   info：包含一个全局性排除（global exclude）文件， 用以放置那些不希望被记录在 `.gitignore` 文件中的忽略模式

-   objects：存储所有数据内容，tree 对象、blob 对象、commit 对象都在这里

-   refs：分支、标签等引用信息，存储指向数据的提交对象指针

-   config：git 项目配置文件，远程仓库信息，用户信息

-   description：仅供 GitWeb 程序使用，无需关心

-   HEAD：当前工作分支的提交对象引用

-   index: 暂存区保存信息

#### 回到项目本身

![image-folder](/posts/coding/git/folder.png)

前面说到，git 保存的不是差异而是快照，相当于某一时刻整个项目下的文件信息，那这个快照又是如何设计实现的呢？

-   目录信息（文件名、目录结构）=> **树（tree）对象**
-   文件内容 => **Blob 对象**
-   提交信息 => **提交对象**

#### `git add [filename|path]`

暂存操作会为每一个文件计算校验和（使用 SHA-1 哈希算法），然后会把当前版本的文件快照保存到 Git 仓库中 （Git 使用 _blob_ 对象来保存它们），最终将校验和加入到暂存区域等待提交

git add 命令会为添加到暂存区的所有文件分别计算校验和，放到 objects 目录下：

git 使用一些底层命令来实现版本控制。

```
git hash-object -w [filename] // 为指定文件计算校验和并写入git数据库
git cat-file -p [hash] // 查看文件对象内容
git cat-file -t [hash] // 查看文件对象类型
```

尝试利用这些底层命令对文件进行简单的版本控制。

文件内容是拿到了，那文件名呢？-- 树对象（tree object）

![data-model-1](/posts/coding/git/data-model-1.png)

git 根据某一时刻暂存区（即 index 文件）的状态创建一个对应的树对象。由此，根据在一段时间内多次修改暂存区便可以得到一系列的树对象。

```
git update-index --add [file] // 更新暂存区
git write-tree // 根据暂存区内容生成树对象
git read-tree --prefix=[dir] [tree-id] // 读取树对象到暂存区
git commit-tree [tree-id] -p [parent-id] -m [message] //
```

#### `git commit -m 'fix: xxxx'`

当使用 `git commit` 进行提交操作时，Git 会先计算每一个子目录的校验和， 然后在 Git 仓库中将这些校验和保存为树对象。随后，Git 便会创建一个提交对象， 它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。 如此一来，Git 就可以在需要的时候重现此次保存的快照。

也就是说，一次提交产生的提交对象包括了所有的文件内容对象的指针和一个树对象的指针

![commit-and-tree](/posts/coding/git/commit-and-tree.png)

#### `git checkout <branch-name>`

分支：指向 **提交对象** 的可变指针，其实就是对某一个 **提交对象** 的引用，由此可见，git 的分支其实是相当“轻量”的。

**HEAD 指针** 用于标识当前所在的本地分支

![head-to-testing](/posts/coding/git/head-to-testing.png)

此时在 testing 分支上进行一次新的提交，可以看到，testing 分支向前移动了，master 分支仍然指向旧的提交对象。

![advance-testing](/posts/coding/git/advance-testing.png)

在文件没有产生变动的情况下，工作区显示的文件内容总是取决于当前分支所指向的提交对象（快照）的内容。

切换分支实际上就是改变 HEAD 指针的指向。

---

### 常用操作&使用技巧

#### Branch

要重命名分支，请使用以下命令：

```shell
git branch -m <old-branch-name> <new-branch-name>
```

要删除分支，请使用以下命令：

```shell
git branch -d <branch-name>
```

删除远程分支

```
git push origin -d <branch-name>
```

清理远端不存在的本地分支的引用

```
git remote prune origin
```

如果分支上有未合并的更改，Git 将阻止您删除它。要强制删除分支，请使用以下命令：

```shell
git branch -D <branch-name>
```

要查看所有分支，请使用以下命令：

```shell
git branch -a
```

要查看所有远程分支，请使用以下命令：

```shell
git branch -r
```

要创建新分支并立即切换到它，请使用以下命令：

```shell
git checkout -b <new-branch-name>
```

#### Commit

提交

```
git commit -m "fix: cccc"
```

要修改最后一次提交的消息或文件，请使用以下命令：

```shell
git commit --amend
```

要修改多个提交，请使用以下命令：

```shell
git rebase -i <commit>
```

要撤销提交，请使用以下命令：

```shell
git revert <commit>
```

#### Stash

git stash 提供一块单独的缓存区用于存储代码片段
git stash 的存储方式是 “栈” **后进先出**

查看缓存列表

```bash
git stash list
```

显示指定历史记录的改动信息

```bash
git stash show [i]
```

将暂未提交到暂存区的改动加入缓存

```bash
git stash
```

从缓存区中取出一条记录，若 i 不提供，则默认取最近的一条缓存记录

```bash
git stash apply [i]
```

#### Tag

git 的标签系统实际上就是 **快照** 机制，一个 tag 记录了在创建这条 tag 前一瞬间整个仓库目录的状态。

要创建标签，请使用以下命令：

```shell
git tag <tag-name>
git tag -a <tag-name> -m "description"
```

要查看所有标签，请使用以下命令：

```shell
git tag
```

要查看标签的详细信息，请使用以下命令：

```shell
git show <tag-name>
```

要删除标签，请使用以下命令：

```shell
git tag -d <tag-name>
```

要推送标签到远程仓库，请使用以下命令：

```shell
git push origin <tag-name>
```

#### Diff

`git diff`命令可以比较工作目录和暂存区域快照之间的差异。也就是修改之后还没有暂存起来的变化内容。

`--staged` 参数可以比较已暂存文件和最后一次提交的文件差异。

```
git diff [--staged]
```

`git difftool` 可以使用你的系统支持的图形化工具来查看比对差异，使用 `git difftool --tool-help` 命令来看你的系统支持哪些 Git Diff 插件。

#### Rebase

Git Rebase 可以用于以下场景：

-   在分支之间同步提交。
-   使提交历史更干净、更直观。
-   将相关提交组合成单个提交。

要在当前分支上变基，请使用以下命令：

```shell
git rebase <branch>
```

![img](/posts/coding/git/rebase.png)

要终止变基，请使用以下命令：

```shell
git rebase --abort
```

在变基过程中如果遇到冲突，需要手动解决。解决冲突后，使用以下命令继续变基：

```shell
git rebase --continue
```

要将分支上的一系列提交合并为单个提交，可以使用以下命令：

```shell
git checkout branch
git rebase -i HEAD~n
```

其中，`n` 是要合并的提交数量。

在编辑器中，将要合并的提交标记为 `squash`，然后保存并关闭编辑器。

`s` 合并， `p` 保留

#### 其他

-   git add/status/commit
-   git pull/push
-   git fetch 拉取远端仓库的改动，不会合并 git pull = git fetch + git merge
-   git push -u [remote]/[branch] 推送到远端仓库并建立分支关联
-   git cherry-pick 将某一次提交的改动应用到当前工作区
-   git restore [--staged] path 撤销改动 --staged 撤销提交(改动还在，只是从暂存区恢复到工作目录)
-   git reset --hard/soft [commit-id] 回退至某个提交 soft: 工作区保留差异 hard: 丢弃这些差异
-   git clean -fd 清除未提交到暂存区的改动和文件
-   git diff [--staged]
-   git branch -a 显示所有分支，包括远端分支 -d 删除分支 -m 修改分支名 大写表示强制（加 -f）
-   git checkout -b [branch name] --track [remote]/[branch-name] 切换到新分支 并与远端分支关联
-   git log -p 显示当前分支的提交记录 -p 可以显示每次提交的改动文件

#### Git Hooks

Git 钩子是在 Git 操作期间自动运行的脚本。它们可以在特定的事件发生时运行，例如提交、合并、推送等等。以下是一些常用的客户端钩子：

**pre-commit** 钩子在执行提交之前运行，可以用来检查代码风格、运行测试等等。要使用 pre-commit 钩子，请创建名为 `.git/hooks/pre-commit` 的文件，并将其内容设置为您想要运行的命令。

**post-commit** 钩子在执行提交后运行，可以用来发送通知、更新文档等等。要使用 post-commit 钩子，请创建名为 `.git/hooks/post-commit` 的文件，并将其内容设置为您想要运行的命令。

**prepare-commit-msg** 钩子在提交消息被编辑器打开之前运行，可以用来自动填充提交消息。要使用 prepare-commit-msg 钩子，请创建名为 `.git/hooks/prepare-commit-msg` 的文件，并将其内容设置为您想要运行的命令。

服务器端钩子在远程仓库被更新时自动运行。以下是一些常用的服务器端钩子：

**pre-receive** 钩子在接收远程分支更新之前运行，可以用来拒绝提交、检查提交消息等等。要使用 pre-receive 钩子，请在服务器上找到仓库的 hooks 目录，并创建名为 `pre-receive` 的文件，并将其内容设置为您想要运行的命令。

**post-receive** 钩子在接收远程分支更新之后运行，可以用来发送通知、更新文档等等。要使用 post-receive 钩子，请在服务器上找到仓库的 hooks 目录，并创建名为 `post-receive` 的文件，并将其内容设置为您想要运行的命令。

#### .gitignore

决定工作目录下哪些文件不被追踪

文件 `.gitignore` 的格式规范如下：

-   所有空行或者以 `#` 开头的行都会被 Git 忽略。
-   可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
-   匹配模式可以以（`/`）开头防止递归。
-   匹配模式可以以（`/`）结尾指定目录。
-   要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（`*`）匹配零个或多个任意字符；`[abc]` 匹配任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 问号（`?`）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。 使用两个星号（``）表示匹配任意中间目录，比如 `a//z`可以匹配`a/z`、`a/b/z`或`a/b/c/z` 等。

例如：

```
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

在这里可以找到更多`ignore`例子：https://github.com/github/gitignore

### git commit 规范

> git commit message 应该遵循 type(scope): subject

**Type**：代表的是提交内容的一种类型，每一种类型都代表着不同的含义，具体的类型取值和含义如下：

1. `feat`：表示开发一个新的需求特性；
2. `fix`：表示修复一个 `bug`；
3. `docs`：表示是针对文档的修改，并没有修改代码；
4. `style`：格式修改，不影响代码功能；
5. `refactor`：不是进行 `feat` 和 `fix` 的代码修改，重构功能；
6. `perf`：提升性能的代码修改；
7. `tes`t：添加测试代码或者修正已经存在的测试功能代码；
8. `build`：修改会影响构建或者依赖的代码；
9. `ci`：修改集成配置的文件或者脚本；
10. `chore`：一些不够影响到源码和测试文件的修改；
11. `revert`：针对之前的一个提交的 `revert` 修改；

**Scope**: 表示的当次 `git` 提交的内容影响的范围，这个范围比较宽泛，比如可以是 `DAO` 层，`Controller` 层，或者是具有特定功能的比如 `utils` 工具模块，权限模块，数据模块等等，只要能跟自己的项目挂上钩，表达出修改的范围就行，如果涉及到的范围比较多的话，可以用 `*` 表示，并不强制要求。

**Subject**: 一段简单的言简意赅的描述，需要体现出本次改动的目的。
