---
title: Git WorkFlow 小记
date: 2018-06-23 12:22:37
categories: 软件开发
tags: [git, workflow]
toc: true
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/22.png" width="45%" height="45%">
什么是 workflow？workflow 就是工作流，即工作的流程，是一个软件过程模型的体现。因为 git 不同分支的交叠，便构成了具有 git 特色的工作流。一个良好的工作流可以让我们的项目历史清晰明了，有利于更好的代码管理。利用 git 的分支管理功能，可以将软件生命周期的各个过程归并到各个分支上，实现软件开发过程中各个操作的隔离。
在项目开发过程中，工作流是一个准则，由开发者自己定义，并自行遵守。本篇文章主要用于督促自己养成一个良好的开发习惯。
<!-- more -->
### 各个分支的使用

Git workflow 分为五个分支，包括 `master`、`develop`、`release`、`hotfix` 和 `feature`，如下图：
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/21.png" width="65%" height="65%">

#### 长期分支

在代码的中央仓库一直存在两个主要分支：`master` 和 `develop`。

其他仓库(非中央仓库)的这两个分支应当始终和中央仓库保持一致，在每次向中央仓库的对应分支合并时，应当先确认中央仓库的对应分支(下面简称中央分支)没有新的提交，如果有新的提交，应当先把本分支的`基`设置为中央分支的最新提交，即使用 `rebase` 将中央分支与本分支合并，再将本分支合并(merge)到中央分支。
- `master` 用于管理发布版本，每次 commit (其他分支向它合并形成的 merge commit)应当对应一个 Tag，也就是形成一个发布版。
- `develop` 用于管理开发版本，所有的开发都会汇总到这个分支。

#### 短期分支

短期分支可以同时存在多个(当然命名不能重复)，每个分支使用完应当被删除掉，包括`release`、`hotfix` 和 `feature`。
- `release` 用于在正式发布之前的预发布版本，在这个版本中的提交都应当是修复 Bug，不能在本分支上开发新的功能。本分支应当从 `develop` 检出，Bug 修复之后合并(merge)到 `develop` 和 `master`。
- `feature` 用于新功能的开发，可以有多个。本分支应当从 `develop` 分支检出，功能开发完成后合并(merge)到 `develop`。
> 在 `release` 和 `feature` 两个分支的开发过程中，如果 `develop` 分支有更新，可以选择不合并 `develop`，如果一定要合并。应当使用 `git rebase` 进行合并，即将 `feature` 的`基`和 `develop` 的最新提交保持一致。

- `hotfix` 用于在版本发布之后的紧急 Bug 修复。本分支应当从 `master` 分支检出，在 Bug 修复之后直接合并(merge)到 `master` 和 `develop`。

### 合并命令

合并命令分为 merge 命令和 rebase 命令，在没有特别说明的情况下的合并命令一般指 merge 命令。

#### Merge 命令

Merge 命令可以让两个分支合并，但可能产生合并提交(merge commit)，在项目中一般都会使用 merge 命令进行分支的合并，但如果在某些情况下不想产生合并提交，则不应该使用这个命令。以将 `feature-1` 合并到 `develop` 为例：
```bash
# 切换到 develop 分支
git checkout develop
# 策略合并 feature 分支
git merge --no-ff feature-1
# 删除原分支
git branch -d feature-1
# 推送 develop 分支到远程仓库
git push origin develop
```

#### Rebase 命令

Rebase 命令和它的字面意思一样，会改变该分支的`基`，它会将该分支的`基`变为另一个分支的最新的提交，`基`是一个分支在另一个分支中分叉后的的第一个提交。rebase 命令不会像 merge 命令那样产生合并提交，它会通过移动一个分支在另一个分支上分叉后的所有提交，形成一个完美的线性历史。例如，在 `feature-1` 的开发过程中需要将 `develop` 合并，但不希望合并提交的产生，便可以使用 rebase 命令：
```bash
# 如果不在 feature-1 分支，切换到 feature-1 分支
git checkout feature-1
# 合并(rebase) develop 分支
git rebase develop
```

如果 `feature` 分支的提交太乱(比如有很多 *Fix bug*)，可以使用交互式 rebase 命令对 `feature` 分支的提交进行重构：
```bash
# 如果不在 feature-1 分支，切换到 feature-1 分支
git checkout feature-1
# 交互式合并(rebase) develop 分支
git rebase -i develop
```
使用 `-i` 参数可以启动交互式的 rebase，它会打开一个文本编辑器，显示所有被移动的提交:
```bash
pick 34b6aca 这是 feature 分支的第一次提交
pick 2bb57ac 修复第一次提交的 Bug
pick 233dc11 添加一个新功能
```
我们可以对这段代码进行编辑：
```bash
pick 34b6aca 这是 feature 分支的第一次提交
fixup 2bb57ac 修复第一次提交的 Bug
pick 233dc11 添加一个新功能
```
这样在最终形成的 `feature` 分支中便不会显示 `2bb57ac` 这次提交了(和之前的提交合并为一个新的提交)。当我们打开交互式 rebase 的时候，在注释里还可以看到其它功能的说明，利用这些功能我们可以随意地更改提交历史

拉取并合并远程分支时使用 rebase 命令可以避免可能产生的合并提交：
```bash
# 采用 rebase 命令拉取并合并远程分支
git pull origin develop --rebase
```
因为 `git pull` 命令是 `git fetch` 命令和 `git merge` 命令的语法糖，加上 `--rebase` 参数会使合并过程采用 rebase 命令合并。

#### 合并提交的产生

合并提交(merge commit)可以将一个分支上的多个提交整合为一个，然后合并到另一个分支。如果两个分支没有出现分叉，这两个分支的合并是不会产生合并提交的。如果出现了分叉，它们的的合并(merge)一定会产生合并提交。

### 合并准则

#### 不能反向合并

从上文我们可以看出，git workflow 中的五个分支是有一定服务关系的，其服务关系如下：
- `feature` -> `develop`
- `release` -> `develop` & `master`
- `develop` -> `master`
- `hotfix` -> `develop` & `master`

在团队协作时，会有一定的服务关系，一般是非中心仓库的分支为中心仓库的分支服务。

这里提到的**不能反向合并**即不能把被服务分支合并(merge)到服务分支(例如不能将 `develop` 合并到 `feature`)。当然，如果在开发过程中一定要反向合并，应当使用 rebase 合并。

#### 采用策略合并

在[Merge 命令](#Merge 命令)中我们使用了 `--no-ff` 参数，这会让 git 的合并(merge)操作不采用 `Fast-Forward` 的合并方式，而是采用策略合并，这样的合并可以保留分支间的合并历史，如下图：
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/20.png" width="65%" height="65%">

#### 在 GitHub 上 Review

GitHub 提供的 Pull Request (简称“PR”)为我们提供了很好的代码合并的工具，开发者可以通过 PR 向自己的仓库或其他协助者的仓库发起合并请求。而且在这个合并请求中，我们可以对每次提交的具体内容和文件的更改情况进行 Review。例如我们可以在 GitHub 上执行 `release` 分支向 `master` 和 `develop` 分支的合并，并且在合并完成后添加发布版本到 GitHub 上。

### 参考文章

1. [简介我的 Git Work Flow](http://zhoulingyu.com/2017/05/08/Git-Work-Flow/)
1. [Git 三大特色之 WorkFlow](https://blog.csdn.net/qq_32452623/article/details/78905181)
