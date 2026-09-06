---
title: git备忘
date: 2023-11-10
summary: git 常用命令
tags:
  - tools
categories: handbook
---

## config

```
git config --list
```

```
git config --global --list
```

```
git config --system --list
```

```
git config --global user.name “[firstname lastname]”
```

```
git config --global user.email “[valid-email]”
```

```sh
git config --global credential.helper 'store'
```

## basic usage

### init, status, diff, add, commit, stash, ignore

```
$ git init
```

```
$ git status
```

```
$ git diff
```

```
$ git add .
```

```
$ git commit -m 'message here'
```

```
$ git commit --amend
```

```
git stash
git checkout branch2
git stash pop
```

新增ignore文件，并不再跟踪

```
# edit .gitignore
git rm -r --cached .
git add .
git commit -m "remove xyz file"
```

### branch

列出分支

```
$ git branch
```

切换分支

```
$ git checkout <branch>
```

创建并切换到新分支:

```
$ git checkout -b <branch>
```

基于当前分支创建新分支：

```
$ git branch <new-branch>
```

基于远程分支创建新的可追溯的分支：

```
$ git branch --track <new-branch> <remote-branch>
```

删除

```
$ git branch -d <branch>
```

```
$ git tag <tag-name>
```

### log

```
$ git log --oneline
```

```
$ git log --author="username"
```

仅显示远端<remote/master>分支与远端<origin/master>分支提交记录的差集：

```
$ git log --oneline <origin/master>..<remote/master> --left-right
```

谁，在什么时间，修改了文件的什么内容：

```
$ git blame <file>
```

### remote

```
$ git remote -v
```

```
$ git remote show <remote>
```

```
$ git remote add <remote> <url>
```

```
$ git fetch <remote>
```

```
git pull --rebase <remote> <branch>
```

```
$ git push remote <remote> <branch>
```

```
git push <remote> --delete <branch> (since Git v1.7.0)
```

```
git remote add <remote name> <url>
git push <remote name> <local branch>:<remote branch>
```

```
$ git push --tags
```

### reset

```
$ git reset --hard HEAD
```

移除缓存区的所有文件（i.e. 撤销上次`git add`）:

```
$ git reset HEAD
```

放弃某个文件的所有本地修改：

```
$ git checkout HEAD <file>
```

重置一个提交（通过创建一个截然不同的新提交）

```
$ git revert <commit>
```

将HEAD重置到指定的版本，并抛弃该版本之后的所有修改：

```
$ git reset --hard <commit>
```

## submodule

使用 `git submodule add <submodule_url>` 命令可以在项目中创建一个子模块。  
此时项目仓库中会多出两个文件：`.gitmodules` 和 `project-sub-1` 。  
通常此时可以使用 `git commit -m "add submodule xxx"` 提交一次，表示引入了某个子模块。提交后，在主项目仓库中，会显示出子模块文件夹，并带上其所在仓库的版本号。

上述步骤在创建子模块的过程中，会自动将相关代码克隆到对应路径，但对于后续使用者而言，对于主项目使用普通的 `clone` 操作并不会拉取到子模块中的实际代码。  
如果希望子模块代码也获取到，一种方式是在克隆主项目的时候带上参数 `--recurse-submodules`，这样会递归地将项目中所有子模块的代码拉取。  
手动更新

```sh
git submodule init
git submodule update
```

1. 当前项目下子模块文件夹内的内容发生了未跟踪的内容变动；
2. 当前项目下子模块文件夹内的内容发生了版本变化；
3. 当前项目下子模块文件夹内的内容没变，远程有更新；

对于第1种情况，通常是在开发环境中，直接修改子模块文件夹中的代码导致的。  
此时在主项目中使用 `git status` 能够看到关于子模块尚
在此情景下，通常需要进入子模块文件夹，按照子模块内部的版本控制体系提交代码。

当提交完成后，主项目的状态则进入了情况2，即当前项目下子模块文件夹内的内容发生了版本变化。  
当子模块版本变化时，在主项目中使用 `git status` 查看仓库状态时，会显示子模块有新的提交。  
在这种情况下，可以使用 `git add/commit` 将其添加到主项目的代码提交中，实际的改动就是那个子模块 `文件` 所表示的版本信息。  
通常当子项目更新后，主项目修改其所依赖的版本时，会产生类似这种情景的 commit 提交信息。

情况3：子模块远程有更新。需要让主项目主动进入子模块拉取新版代码，进行升级操作。

```sh
cd project-sub-1
git pull origin master
```

子模块目录下的代码版本会发生变化，转到情况2的流程进行主项目的提交。  
当主项目的子项目特别多时，可能会不太方便，此时可以使用 `git submodule` 的一个命令 `foreach` 执行：

```
git submodule foreach 'git pull origin master'
```

使用 `git submodule deinit` 命令卸载一个子模块。这个命令如果添加上参数 `--force`，则子模块工作区内即使有本地的修改，也会被移除。

```
git submodule deinit project-sub-1
git rm project-sub-1
```

此时，主项目中关于子模块的信息基本已经删除，可以提交代码：

```
git commit -m "delete submodule project-sub-1"
```

## worktree

一般在主目录外建立另一个worktree，这样可以同时处理两个分支

```
usage: git worktree add [-f] [--detach] [--checkout] [--lock [--reason <string>]]
                        [--orphan] [(-b | -B) <new-branch>] <path> [<commit-ish>]
   or: git worktree list [-v | --porcelain [-z]]
   or: git worktree lock [--reason <string>] <worktree>
   or: git worktree move <worktree> <new-path>
   or: git worktree prune [-n] [-v] [--expire <expire>]
   or: git worktree remove [-f] <worktree>
   or: git worktree repair [<path>...]
   or: git worktree unlock <worktree>
```

For example: `git worktree add ../tree`

## 清理

清理

```
git clean -xdf
```

其中`-x` remove ignored files，`d` remove untracked directories, `-f` remove untracked nested repositories

## reference

1. [Arabic Git Cheat Sheet](https://github.com/arslanbilal/git-cheat-sheet/blob/master/other-sheets/git-cheat-sheet-ar.md)
2. [Chinese Git Cheat Sheet](https://github.com/arslanbilal/git-cheat-sheet/blob/master/other-sheets/git-cheat-sheet-zh.md)
3. [Hindi Git Cheat Sheet](https://github.com/arslanbilal/git-cheat-sheet/blob/master/other-sheets/git-cheat-sheet-hi.md)
4. [Turkish Git Cheat Sheet](https://github.com/arslanbilal/git-cheat-sheet/blob/master/other-sheets/git-cheat-sheet-tr.md)
5. [Spanish Git Cheat Sheet](https://github.com/arslanbilal/git-cheat-sheet/blob/master/other-sheets/git-cheat-sheet-es.md)
