---
title: 使用 git bisect 二分法快速定位问题
date: 2024-09-10 11:55:20
categories: 技术
tags: [git]
toc: true
reward: true
---

<img src="http://home.cs-tao.cc/github-content/contents/blog/image/git-bisect.jpeg?1" width="36%" height="36%">

最近作为「内包」支援公司内的另一个项目，需要和另一个团队的同事们合作，不熟悉他们的项目，在经过两周的合作后，某个不常用的页面终于打不开了。

这个时候我们便可以使用 git 自带的二分查找工具 `git bisect` 来快速定位出问题的提交，找相关人员排查问题。

<!-- more -->

`git bisect` 最常用的命令有如下三个：

```bash
# 开始(重置)二分查找流程
❯ git bisect start

# 标记当前(或指定)提交为有问题的提交
❯ git bisect bad [<rev>]

# 标记当前(或指定)提交为正常的提交
❯ git bisect good [<rev>]
```

下面先记录我是如何用这三个命令快速定位问题的，然后会介绍一下 bisect 其它命令和其使用场景。

## 问题定位

1. 首先使用你熟悉的 git 工具查看一下提交历史，我使用的是 [tig](https://github.com/jonas/tig)，提交历史如下

   ```bash
   o [HEAD] [eacd4a0a85bfcc8691f22368079acab7412575d7] feat: ********
   M─┐ Merge branch '********' into '********'
   │ o [66504f3e0b73b568d379adbaf0e7b031d595187e] feat: ********
   o─┘ [58625437bda334df576eb5b5031fba7ec990be65] fix: ********
   o [07ee37373716bfb75f391ab3b65f68640a5e3951] feat: ********
   o [ee948eb50bfcf7bfe47790f50053439f05ac0e5a] feat: ********
   o [fedd7e523f90679ae0048138f5cf21ebf07be733] fix: ********
   o [8ad08f09eb3a3225af9c05e59210c99ecd53f330] fix: ********
   o [f71b6535d88613cb08211a277ce7f2c92c83d00e] feat: ********
   o [e160869ec18a1980e100acc07d81a385392a8039] feat: ********
   o [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   o [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ********
   o [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********
   o [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********
   o [f3517468c23495063f99a9e49c657745d95ba88f] chore: ********
   o [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********
   o [839367048641d147cd28a38991a5c6d7dca3bb0d] feat: ********
   o [147833f0a4f4ed7de21b62d919e937c4c6ec6cc4] feat: ********
   o [43835476244b330ab8bd119e94c45aeaf0f32afa] chore: ********
   o [e0c12e65ec86db44557756ddfc2ad510878ea9bf] feat: ********
   M─┐ [ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'
   │ o [5cd65c0fd7ee8309d6ddec38e232eeae19f93a03] chore: 发布正式版本
   ```

1. 接下来执行 `git bisect start` 命令，进入二分查找模式，并标记最新的提交为有问题的提交

   ```bash
   ❯ git bisect start
   status: waiting for both good and bad commits

   ❯ git bisect bad
   status: waiting for good commit(s), bad commit known
   ```

1. 根据提示，接下来我们需要标记正常的提交。我们审视一下上面的提交历史，可以发现 `[ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'` 看起来是上次发布上线的版本，我们切到这个提交上测试一下

   ```bash
   ❯ git checkout ee62949cc48a5f60b16ef5fa62d0418ee36ac855
   Note: switching to 'ee62949cc48a5f60b16ef5fa62d0418ee36ac855'
   ```

   发现页面能正常打开，所以我们可以标记此提交是正常的

   ```bash
   ❯ git bisect good
   Bisecting: 10 revisions left to test after this (roughly 3 steps)
   [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   ```

   此时的提交历史检视结果如下（同时可以关注到 git 已经自动切换到了 `79a20e29636178e33479ec35cd23101b4080a6aa` 提交）

   ```bash
   ❌ o [HEAD] [eacd4a0a85bfcc8691f22368079acab7412575d7] feat: ********
   ❔ M─┐ Merge branch '********' into '********'
   ❔ │ o [66504f3e0b73b568d379adbaf0e7b031d595187e] feat: ********
   ❔ o─┘ [58625437bda334df576eb5b5031fba7ec990be65] fix: ********
   ❔ o [07ee37373716bfb75f391ab3b65f68640a5e3951] feat: ********
   ❔ o [ee948eb50bfcf7bfe47790f50053439f05ac0e5a] feat: ********
   ❔ o [fedd7e523f90679ae0048138f5cf21ebf07be733] fix: ********nv
   ❔ o [8ad08f09eb3a3225af9c05e59210c99ecd53f330] fix: ********
   ❔ o [f71b6535d88613cb08211a277ce7f2c92c83d00e] feat: ********
   ❔ o [e160869ec18a1980e100acc07d81a385392a8039] feat: ********
   ❔ o [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   ❔ o [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ********
   ❔ o [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********
   ❔ o [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********
   ❔ o [f3517468c23495063f99a9e49c657745d95ba88f] chore: ********
   ❔ o [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********
   ❔ o [839367048641d147cd28a38991a5c6d7dca3bb0d] feat: ********
   ❔ o [147833f0a4f4ed7de21b62d919e937c4c6ec6cc4] feat: ********
   ❔ o [43835476244b330ab8bd119e94c45aeaf0f32afa] chore: ********
   ❔ o [e0c12e65ec86db44557756ddfc2ad510878ea9bf] feat: ********
   ✅ M─┐ [ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'
   │ o [5cd65c0fd7ee8309d6ddec38e232eeae19f93a03] chore: 发布正式版本
   ```

   继续检查当前提交是否正常，如果正常，则执行 `git bisect good`，如果不正常，则执行 `git bisect bad`，直至找到首个有问题的提交为止

   > 每次检查当前提交是否正常之前需要执行一些准备命令确保外部依赖被正确安装。比如前端项目中一般需要执行 `npm install` 命令来更新依赖

1. 标记当前提交 `79a20e29636178e33479ec35cd23101b4080a6aa` 是有问题的

   ```bash
   ❯ git bisect bad
   Bisecting: 4 revisions left to test after this (roughly 2 steps)
   [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********

   # 查找结果
   ❌ o [HEAD] [eacd4a0a85bfcc8691f22368079acab7412575d7] feat: ********
   ❌ M─┐ Merge branch '********' into '********'
   ❌ │ o [66504f3e0b73b568d379adbaf0e7b031d595187e] feat: ********
   ❌ o─┘ [58625437bda334df576eb5b5031fba7ec990be65] fix: ********
   ❌ o [07ee37373716bfb75f391ab3b65f68640a5e3951] feat: ********
   ❌ o [ee948eb50bfcf7bfe47790f50053439f05ac0e5a] feat: ********
   ❌ o [fedd7e523f90679ae0048138f5cf21ebf07be733] fix: ********
   ❌ o [8ad08f09eb3a3225af9c05e59210c99ecd53f330] fix: ********
   ❌ o [f71b6535d88613cb08211a277ce7f2c92c83d00e] feat: ********
   ❌ o [e160869ec18a1980e100acc07d81a385392a8039] feat: ********
   ❌ o [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   ❔ o [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ********
   ❔ o [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********
   ❔ o [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********
   ❔ o [f3517468c23495063f99a9e49c657745d95ba88f] chore: ********
   ❔ o [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********
   ❔ o [839367048641d147cd28a38991a5c6d7dca3bb0d] feat: ********
   ❔ o [147833f0a4f4ed7de21b62d919e937c4c6ec6cc4] feat: ********
   ❔ o [43835476244b330ab8bd119e94c45aeaf0f32afa] chore: ********
   ❔ o [e0c12e65ec86db44557756ddfc2ad510878ea9bf] feat: ********
   ✅ M─┐ [ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'
   │ o [5cd65c0fd7ee8309d6ddec38e232eeae19f93a03] chore: 发布正式版本
   ```

1. 标记当前提交 `f11d08852b7fb0ef7e9d16056280edac5a772d57` 是正常的

   ```bash
   ❯ git bisect good
   Bisecting: 2 revisions left to test after this (roughly 1 step)
   [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********

   # 查找结果
   ❌ o [HEAD] [eacd4a0a85bfcc8691f22368079acab7412575d7] feat: ********
   ❌ M─┐ Merge branch '********' into '********'
   ❌ │ o [66504f3e0b73b568d379adbaf0e7b031d595187e] feat: ********
   ❌ o─┘ [58625437bda334df576eb5b5031fba7ec990be65] fix: ********
   ❌ o [07ee37373716bfb75f391ab3b65f68640a5e3951] feat: ********
   ❌ o [ee948eb50bfcf7bfe47790f50053439f05ac0e5a] feat: ********
   ❌ o [fedd7e523f90679ae0048138f5cf21ebf07be733] fix: ********
   ❌ o [8ad08f09eb3a3225af9c05e59210c99ecd53f330] fix: ********
   ❌ o [f71b6535d88613cb08211a277ce7f2c92c83d00e] feat: ********
   ❌ o [e160869ec18a1980e100acc07d81a385392a8039] feat: ********
   ❌ o [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   ❔ o [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ********
   ❔ o [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********
   ❔ o [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********
   ❔ o [f3517468c23495063f99a9e49c657745d95ba88f] chore: ********
   ✅ o [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********
   ✅ o [839367048641d147cd28a38991a5c6d7dca3bb0d] feat: ********
   ✅ o [147833f0a4f4ed7de21b62d919e937c4c6ec6cc4] feat: ********
   ✅ o [43835476244b330ab8bd119e94c45aeaf0f32afa] chore: ********
   ✅ o [e0c12e65ec86db44557756ddfc2ad510878ea9bf] feat: ********
   ✅ M─┐ [ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'
   │ o [5cd65c0fd7ee8309d6ddec38e232eeae19f93a03] chore: 发布正式版本
   ```

1. 标记当前提交 `782c1fa6852522d5db16cfff88c7051c2ac0bf09` 是正常的

   ```bash
   ❯ git bisect good
   Bisecting: 0 revisions left to test after this (roughly 1 step)
   [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ********

   # 查找结果
   ❌ o [HEAD] [eacd4a0a85bfcc8691f22368079acab7412575d7] feat: ********
   ❌ M─┐ Merge branch '********' into '********'
   ❌ │ o [66504f3e0b73b568d379adbaf0e7b031d595187e] feat: ********
   ❌ o─┘ [58625437bda334df576eb5b5031fba7ec990be65] fix: ********
   ❌ o [07ee37373716bfb75f391ab3b65f68640a5e3951] feat: ********
   ❌ o [ee948eb50bfcf7bfe47790f50053439f05ac0e5a] feat: ********
   ❌ o [fedd7e523f90679ae0048138f5cf21ebf07be733] fix: ********
   ❌ o [8ad08f09eb3a3225af9c05e59210c99ecd53f330] fix: ********
   ❌ o [f71b6535d88613cb08211a277ce7f2c92c83d00e] feat: ********
   ❌ o [e160869ec18a1980e100acc07d81a385392a8039] feat: ********
   ❌ o [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   ❔ o [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ********
   ❔ o [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********
   ✅ o [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********
   ✅ o [f3517468c23495063f99a9e49c657745d95ba88f] chore: ********
   ✅ o [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********
   ✅ o [839367048641d147cd28a38991a5c6d7dca3bb0d] feat: ********
   ✅ o [147833f0a4f4ed7de21b62d919e937c4c6ec6cc4] feat: ********
   ✅ o [43835476244b330ab8bd119e94c45aeaf0f32afa] chore: ********
   ✅ o [e0c12e65ec86db44557756ddfc2ad510878ea9bf] feat: ********
   ✅ M─┐ [ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'
   │ o [5cd65c0fd7ee8309d6ddec38e232eeae19f93a03] chore: 发布正式版本
   ```

1. 标记当前提交 `fbdc27d4498e4d27828e8b7037d68c73f272878d` 是有问题的

   ```bash
   ❯ git bisect bad
   Bisecting: 0 revisions left to test after this (roughly 0 steps)
   [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********

   # 查找结果
   ❌ o [HEAD] [eacd4a0a85bfcc8691f22368079acab7412575d7] feat: ********
   ❌ M─┐ Merge branch '********' into '********'
   ❌ │ o [66504f3e0b73b568d379adbaf0e7b031d595187e] feat: ********
   ❌ o─┘ [58625437bda334df576eb5b5031fba7ec990be65] fix: ********
   ❌ o [07ee37373716bfb75f391ab3b65f68640a5e3951] feat: ********
   ❌ o [ee948eb50bfcf7bfe47790f50053439f05ac0e5a] feat: ********
   ❌ o [fedd7e523f90679ae0048138f5cf21ebf07be733] fix: ********
   ❌ o [8ad08f09eb3a3225af9c05e59210c99ecd53f330] fix: ********
   ❌ o [f71b6535d88613cb08211a277ce7f2c92c83d00e] feat: ********
   ❌ o [e160869ec18a1980e100acc07d81a385392a8039] feat: ********
   ❌ o [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   ❌ o [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ********
   ❔ o [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********
   ✅ o [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********
   ✅ o [f3517468c23495063f99a9e49c657745d95ba88f] chore: ********
   ✅ o [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********
   ✅ o [839367048641d147cd28a38991a5c6d7dca3bb0d] feat: ********
   ✅ o [147833f0a4f4ed7de21b62d919e937c4c6ec6cc4] feat: ********
   ✅ o [43835476244b330ab8bd119e94c45aeaf0f32afa] chore: ********
   ✅ o [e0c12e65ec86db44557756ddfc2ad510878ea9bf] feat: ********
   ✅ M─┐ [ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'
   │ o [5cd65c0fd7ee8309d6ddec38e232eeae19f93a03] chore: 发布正式版本
   ```

1. 标记当前提交 `83900f6d009f0158dfce0fccc241b2fa013d1743` 是正常的

   ```bash
   ❯ git bisect good
   fbdc27d4498e4d27828e8b7037d68c73f272878d is the first bad commit
   commit fbdc27d4498e4d27828e8b7037d68c73f272878d
   Author: ******** <********@********.com>
   Date:   Thu Sep 5 15:11:00 2024 +0800

       feat: ********

   .../********.tsx                           |   5 +
   src/********.js                            |   4 +
   src/********.d.ts                          |   5 +
   src/********.ts                            |  10 ++
   .../********.ts                            |   2 +
   src/********.ts                            |  21 ++++
   src/********.ts                            |  36 ++++--
   .../********.test.ts                       | 114 ++++++++++++++++++
   src/********.ts                            |  88 ++++++++++++++
   .../********.ts                            | 133 ++++++++++++++++++---
   src/********.ts                            |  47 ++++++--
   src/********.ts                            |   6 +
   src/********.ts                            |   2 +
   src/********.ts                            |  78 ++++++++++++
   src/********.tsx                           |   2 +
   src/********.tsx                           |   5 +
   16 files changed, 524 insertions(+), 34 deletions(-)
   create mode 100644 src/********.ts
   create mode 100644 src/********.ts
   create mode 100644 src/********.test.ts
   create mode 100644 src/********.ts
   create mode 100644 src/********.ts
   create mode 100644 src/********.ts

   # 查找结果
   ❌ o [HEAD] [eacd4a0a85bfcc8691f22368079acab7412575d7] feat: ********
   ❌ M─┐ Merge branch '********' into '********'
   ❌ │ o [66504f3e0b73b568d379adbaf0e7b031d595187e] feat: ********
   ❌ o─┘ [58625437bda334df576eb5b5031fba7ec990be65] fix: ********
   ❌ o [07ee37373716bfb75f391ab3b65f68640a5e3951] feat: ********
   ❌ o [ee948eb50bfcf7bfe47790f50053439f05ac0e5a] feat: ********
   ❌ o [fedd7e523f90679ae0048138f5cf21ebf07be733] fix: ********
   ❌ o [8ad08f09eb3a3225af9c05e59210c99ecd53f330] fix: ********
   ❌ o [f71b6535d88613cb08211a277ce7f2c92c83d00e] feat: ********
   ❌ o [e160869ec18a1980e100acc07d81a385392a8039] feat: ********
   ❌ o [79a20e29636178e33479ec35cd23101b4080a6aa] feat: ********
   ❌ o [fbdc27d4498e4d27828e8b7037d68c73f272878d] feat: ******** <-- 首个有问题的提交❗️❗️❗️
   ✅ o [83900f6d009f0158dfce0fccc241b2fa013d1743] fix: ********
   ✅ o [782c1fa6852522d5db16cfff88c7051c2ac0bf09] feat: ********
   ✅ o [f3517468c23495063f99a9e49c657745d95ba88f] chore: ********
   ✅ o [f11d08852b7fb0ef7e9d16056280edac5a772d57] feat: ********
   ✅ o [839367048641d147cd28a38991a5c6d7dca3bb0d] feat: ********
   ✅ o [147833f0a4f4ed7de21b62d919e937c4c6ec6cc4] feat: ********
   ✅ o [43835476244b330ab8bd119e94c45aeaf0f32afa] chore: ********
   ✅ o [e0c12e65ec86db44557756ddfc2ad510878ea9bf] feat: ********
   ✅ M─┐ [ee62949cc48a5f60b16ef5fa62d0418ee36ac855] Merge branch '********' into 'release-24-09-02'
   │ o [5cd65c0fd7ee8309d6ddec38e232eeae19f93a03] chore: 发布正式版本
   ```

## 其它命令

你只需知道 `git bisect help` 命令就够了 💯

```bash
❯ git bisect help
usage: git bisect [help|start|bad|good|new|old|terms|skip|next|reset|visualize|view|replay|log|run]

git bisect help
        print this long help message.
git bisect start [--term-{new,bad}=<term> --term-{old,good}=<term>]
                 [--no-checkout] [--first-parent] [<bad> [<good>...]] [--] [<pathspec>...]
        reset bisect state and start bisection.
git bisect (bad|new) [<rev>]
        mark <rev> a known-bad revision/
                a revision after change in a given property.
git bisect (good|old) [<rev>...]
        mark <rev>... known-good revisions/
                revisions before change in a given property.
git bisect terms [--term-good | --term-bad]
        show the terms used for old and new commits (default: bad, good)
git bisect skip [(<rev>|<range>)...]
        mark <rev>... untestable revisions.
git bisect next
        find next bisection to test and check it out.
git bisect reset [<commit>]
        finish bisection search and go back to commit.
git bisect (visualize|view)
        show bisect status in gitk.
git bisect replay <logfile>
        replay bisection log.
git bisect log
        show bisect log.
git bisect run <cmd>...
        use <cmd>... to automatically bisect.

Please use "git help bisect" to get the full man page.
```

这些命令都比较简单，但必须重点提一下的是 `git bisect run <cmd>...` 命令，在标记了 good 和 bad 提交之后，可以通过此命令自动查找出问题的提交。

举一个前端常见的 🌰：**依赖安装失败**。前端项目免不了进行依赖安装，如果依赖安装失败后面的工作几乎无法开展。

当遇到最新的代码无法安装依赖，但历史版本可以安装时：我们可以标记最新的提交为 bad，标记某个历史提交为 good，然后执行 `git bisect run npm i`，接下来 git 便会**自动**以二分顺序切出各个提交，逐一执行 `npm i` 命令。如果命令以非 0 退出，则标记该提交为 bad，反之标记为 good，直至找到首次出问题的提交。

## 思路扩展

同样的思路，在项目实践中，我们可以将主分支上的每一个提交都部署到测试环境，当出现一些不好定位的问题时，通过类似的工具去标记「正常的版本」和「有问题的版本」，结合人工检测或脚本检测（类似于 e2e 测试脚本）来自动查找首次出问题的版本。
