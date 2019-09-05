# 【GIT】GIT使用中的一些实用技巧

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [关于 HEAD](#关于-head)
- [我只想看看远端的代码长什么样子了 fetch](#我只想看看远端的代码长什么样子了-fetch)
- [菠萝菠萝蜜](#菠萝菠萝蜜)
  - [回退代码 revert 和 reset](#回退代码-revert-和-reset)
- [提交时的“骚”操作](#提交时的骚操作)
  - [改写最后一次提交 commit --amend](#改写最后一次提交-commit-amend)
  - [汇合提交 rebase -i](#汇合提交-rebase-i)
  - [修改提交 rebase -i](#修改提交-rebase-i)
- [玩转分支](#玩转分支)
  - [合并分支 merge 和 rebase](#合并分支-merge-和-rebase)
    - [merge](#merge)
    - [rebase](#rebase)
  - [提取提交 cherry-pick](#提取提交-cherry-pick)
  - [汇合分支的提交然后合并到指定分支](#汇合分支的提交然后合并到指定分支)
- [参考](#参考)

<!-- /code_chunk_output -->

## 关于 HEAD

*HEAD~~* 或 *HEAD~2* 表示前两次修改

## 我只想看看远端的代码长什么样子了 fetch

执行 *pull*，远程数据库的内容就会自动合并。但是，有时只是想确认本地数据库的内容而不想合并。这种情况下，请使用 *fetch*。
执行 *fetch* 就可以取得远程数据库的最新历史记录。取得的提交会导入到没有名字的分支，这个分支可以从名为 FETCH_HEAD 的退出。

## 菠萝菠萝蜜

### 回退代码 revert 和 reset

- *revert*  取消提交，执行后， 会生成一个 Revert xxx 内容的提交
- *reset*  遗弃提交，所以它不会生成一次新的提交

*reset* 有三种模式：

| 模式  | HEAD的位置 |  索引  | 工作树 |
| ----- | ---------- | ------ | ------ |
| soft  | 修改       | 不修改 | |
| mixed | ^       | 修改   | 不修改 |
| hard  | ^       | 修改   |    |

如果 *reset* 错了，可以通过

```shell
git reset --hard ORIG_HEAD
```

还原到 *reset* 前的状态。

TODO: 我可以指定 revert 任意一次提交吗? revert的内容是仅那次提交的修改，还是那次提交之后所有的修改？

**问：** 对于已经发布的提交，reset后是如何在MR中体现的?  
**答：** reset 后再 commit，MR只保留reset后的 commit  

## 提交时的“骚”操作

### 改写最后一次提交 commit --amend

有时难免粗心大意，commit 后又发现代码中又一些小问题，不想再为了这么个小改动而多出来一次提交纪录！这时候可以通过 *--amend* 将**最后**一次提交的内容给修改掉！

```shell
git add .
git commit --amend
```

### 汇合提交 rebase -i

比如说

```shell
git rebase -i HEAD~~
```

然后会打开一个文本编辑器，会显示 HEAD 到 HEAD~~ 的提交  
将 HEAD commit 前面的 pick 改成 squash  
保存退出  
然后会自动合并后提交，接着会显示提交信息的编辑器，编辑信息后保存退出  
这样两个提交就汇合成一个提交了  

### 修改提交 rebase -i

前面的 commit --amend 只能修改最后一次提交，但是使用 rebase -i 我们可以修改任意一次提交!  
首先选择要修改的提交，比如前两次的提交  

```shell
git rebase -i HEAD~~
```

然后将第一行的 pick 改成 eidt，保存退出  
这个时候修改过的提交会呈现退出状态  
然后打开要修改的文件，编辑  
再用  

```shell
git add .
git commit --amend
```

保存修改  
要改的提交都改完了的话，执行  

```shell
git rebase --continue
```

这个时候很可能出现冲突，需要修改冲突后再执行 *add* 和 *rebase --continue*

如果想要还原到 rebase 之前的状态

```shell
git reset --hard ORIG_HEAD
```

## 玩转分支

### 合并分支 merge 和 rebase

*merge* 和 *rebase* 都可以合并分支的代码，主要差异在于合并后的历史纪录  
个人比较喜欢 *rebase* ，生成的历史纪录更简洁易懂  
比如说有两个分支 master 和 fix  

#### merge

*merge* fix 和 master 两个分支的修改会生成一个提交，保持了修改内容的纪录，但是修改纪录很复杂

```shell
git checkout master
git merge fix
```

解决冲突

```shell
git commit . -m "合并fix分支到master"
```

#### rebase

*rebase* fix 分支的修改会添加在 master 后面，这样可能会有冲突，但处理之后，历史纪录是一条直线，非常好懂

```shell
git checkout fix
git rebase master
```

解决冲突

```shell
git add .
git rebase --continue
```

*rebase* 的时候，修改冲突后的提交不是使用 *commit* 命令，而是执行 *rebase --continue*。若要取消 *rebase*，指定 *--abort*。

这个时候， master 的最后一次修改后面，又追加了 fix 的修改，
然后切换到 master 分支后合并，将 master 分支的 HEAD 挪到后面加入的 fix 的修改

```shell
git checkout master
git merge fix
```

**建议：**

- 在 master 分支中更新 fix 分支的代码，使用 *rebase*
- 在 fix 分支中导入 master 分支的修改，先使用 *rebase*，再使用 *merge*

TODO: 没看懂建议...

### 提取提交 cherry-pick

如果我代码的修改上错了分支，或者想将分支的修改导入到另一个分支，可以使用 *cherry-pick*
比如我想将某个分支上的 提交 1ab2 导入到  master

```shell
git checkout master
git cherry-pick 1111
git add
git commit
```

TODO: **问：** 只能导入那一次提交的内容吗？比如 master 上有一次提交了，fix 分支上有两次提交，cherry-pick fix上的第二次提交，那么fix上的第一次提交会否导入？

### 汇合分支的提交然后合并到指定分支

```shell
git checkout master
git merge --squash fix
```

修改冲突

```shell
git commit . -m "合并fix分支上的所有提交到master分支"
```

## 参考

[猴子都能懂的GIT入门](https://backlog.com/git-tutorial/cn/reference)  
[Git - 叹为观止的 log 命令 & 其参数](https://blog.csdn.net/qq_32452623/article/details/79599503)
