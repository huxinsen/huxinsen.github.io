---
layout: post
title: 如何设置 Git 新建分支的快捷命令
description: '介绍自动添加分支前缀和基于当前分支名创建新分支的方法'
share: false
tags: [Git, Shell]
image:
  feature: abstract-7.jpg
---

开发中经常使用 git 命令，写多了就想提高点效率，查到可以通过 `.gitconfig` 设置别名，比如 co 代表 checkout，后来进一步发现 Oh My Zsh 已经内置了很多 git 命令的缩写，输入效率又提高了！随后琢磨着怎么自动给新分支加前缀以及基于当前分支名创建新分支，于是有了这篇设置指南。

## 零、基础别名设置

### 1. .gitconfig 设置

添加如下配置后，输入 `git co branch_name` 就达到和输入 `git checkout branch_name` 一样的效果。

```sh
[alias]
	co = checkout
	br = branch
	ci = commit
	st = status
	cp = cherry-pick
```

### 2. Oh My Zsh 内置 git 命令

我常用的有：

```sh
alias gb='git branch'
alias gco='git checkout'
alias gcp='git cherry-pick'
alias gl='git pull'
alias glg='git log --stat'
alias gm='git merge'
alias gp='git push'
alias gst='git status'
alias gsta='git stash push'
alias gstp='git stash pop'
```

可以查看[所有的配置](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/git/git.plugin.zsh)。有一些别名我觉得长或不好记就改写了，比如：

```sh
alias gcm='git commit -m'
alias gcmd='git commit --amend'
alias grsth='git reset --hard'
```

## 一、自动添加分支前缀

通常需求都有对应的 Jira ID，如 `SPCB-1234`，我们在建开发分支时就会创建 `feature/SPCB-1234` 的分支。在 `.gitconfig` 添加下面的别名：

```sh
[alias]
	cbf = "!f() { git checkout -b feature/$1; }; f"
```

并在 `~/.zshrc` 里加入：

```sh
alias gcbf='git cbf'
```

就可以实现输入 `gcbf SPCB-1234` 来创建 `feature/SPCB-1234` 分支并进行切换了。

> Note: 使用 `git config --global alias.co checkout` 这样的命令和编辑 `.gitconfig` 文件的 alias 两者效果是一样的，需要注意的是，如果别名以 `!` 开头那么内容就会被当作 Shell 命令。

这里顺便学习一下 Shell。
在 Shell 中，调用函数时可以向其传递参数。在函数体内部，通过 `$n` 的形式来获取参数的值，例如，`$1` 表示第一个参数，`$2` 表示第二个参数... 那上面的加前缀的别名就很容易理解了。

同时设置 `.gitconfig` 和 `~/.zshrc` 的方法有点麻烦，不如直接在 `~/.zshrc` 定义函数来得简单：

```sh
gcbf() { git checkout -b feature/$1 }
```

或者，我们也可以在别名里定义函数：

```sh
alias gcbf='f() { git checkout -b feature/$1; };f'
```

其实也没必要添加命名函数，使用匿名函数即可：

```sh
alias gcbf='(){ git checkout -b feature/$1; }'
```

## 二、基于当前分支名创建新分支

### 1. 分支管理

我所在团队的代码仓库主要分为 `feature/*`、`hotfix/*`、`test`、`uat`、`master` 和 `release` 分支，`*` 代表 Jira ID，一般按如下规范管理：

- 提测时 `feature/*` 分支合入 `test` 分支，部署 TEST 环境由 QA 进行测试。
- QA 测试通过后，`feature/*` 分支合入 `uat` 分支，部署 UAT 环境给 Local 人员测试。
- UAT 测试结束后，`feature/*` 分支合入 `master` 分支。
- 最后 QA 将 `master` 分支合入 `release` 分支，部署 LIVE 环境，正式发布。
- 开发通常基于 `master` 分支创建 `feature/*` 分支，基于 `release` 分支创建 `hotfix/*` 分支。

有时候个别需求需要单独发版，也会直接从 `feature/*` 分支合入 `release` 分支，所以开发一个需求至少需要合三四次代码。对于合代码，一般会新建 `merge/*` 分支来解决冲突，防止污染 `feature/*` 分支。

### 2. 创建 `merge/*` 分支

有了前面自动添加分支前缀的方法，我们可以自动添加 merge 前缀了，但是大多时候我们会处在 `feature/*` 分支开发，合代码时候需要创建或切换 merge 分支，比如，我们在 `feature/SPCB-1234` 分支，需要新建一个 merge 分支合代码到 test，分支名是 `merge/SPCB-1234/test`。这时候手动输入很麻烦，能不能用命令实现快速切换呢？

现在的关键点是如何复用 Jira ID，看下面的函数：

```sh
gcbmt() {
  local current_branch=$(git_current_branch)
  local arr=(`echo ${current_branch//\// }`)
  git checkout -b merge/${arr[@]:1:1}/test
}
```

- 使用 `$()` 或 `` ` ` `` 包裹表达式。
- `git_current_branch` 是 Oh My Zsh 内置的命令，使用 `git symbolic-ref --short HEAD` 效果是一样的。
- `${current_branch//\// }` 进行字符串替换，通过 `${string//substring/replacement}` 把斜杠换成空格。zsh 需要加上反斜杠进行转义，bash 不需要。
- 使用 `echo` 可以返回任何字符串结果。
- Shell 数组用括号来表示，元素用"空格"符号分割开，语法格式为：
  `array_name=(value1 value2 ... valueN)`。

  实验中发现 `arr=(${current_branch//\// })` 对 bash 是可以的，但对 zsh 不生效，原因是 zsh 会解释成 `arr=("feature SPCB-1234")`（尽管我们没有明确指明这对 `"`），使用 `echo`，就可以解释成 `arr=(feature SPCB-1234)` 了。

- bash 数组索引从 0 开始，zsh 如果不设置 `KSH_ARRAYS` 选项的话，是从 1 开始。所以为了统一行为，使用 `${array[@]:offset:length}` 来取值。命令中的 `arr[@]` 获取数组中的所有元素，中间的 1 是偏移量（总是从 0 开始），后面的 1 是想要的元素个数。

现在我们可以快速地创建或切换 merge 分支了。`gcbmt/gcbmu/gcbmm/gcbmr` 分别创建并转到 `test/uat/master/release` 对应的 merge 分支，类似地，可以使用 `gcomt/gcomu/gcomm/gcomr` 切换到对应的分支。基本的切换流程如下图：

![分支切换](/images/2022-03-28-git-branch-alias/checkout-branch.svg)

## 三、分支合并

我们在 feature 分支开发时，master 分支经常会有一些更新，如果我们依赖这些更新的话，这时候需要切换到 master 拉取最新代码，然后再合并到 feature 分支上。这一系列操作可以封装成一个命令实现自动化：

```sh
gum() {
  local branch=$(git_current_branch)
  git checkout master
  git pull
  git checkout $branch
  git merge master
}
```

这里 `gum` 代表“git update master”，可以随便起别的名字。同理，还有 `gut/guu/gur`。自动拉取更新可以避免手动情况下没拉取最新代码导致的一些冲突和错误。

有时候我们在 feature 分支修复一个 test 环境的 bug 后，要合到 test 分支给 QA 验收。这时候我们使用 `gcomt` 切换到 test 的 merge 分支，接着需要合并 feature 的更新到这个 merge 分支。这里可以使用下面的 `gmf` 命令更方便地合并：

```sh
gmf() {
  local current_branch=$(git_current_branch)
  local arr=(`echo ${current_branch//\// }`)
  git merge feature/${arr[@]:1:1}
}
```

到这里可以看出获取 Jira ID 是一个通用的操作，可以封装成一个函数：

```sh
# 获取 feature Jira ID
fid() {
  local current_branch=$(git_current_branch)
  local arr=(`echo ${current_branch//\// }`)
  echo ${arr[@]:1:1}
}
```

那么 `gmf` 可以简化成：

```sh
alias gmf='(){ git merge feature/$(fid); }'
```

`gcomt` 和 `gmf` 可以进一步一起使用：

```sh
# 切换 merge test 分支并合并 feature 更新
alias guft='gcomt && gmf'
```

分支合并的流程图如下所示，加了括号的命令表示在 `feature/*` 或 `hotfix/*` 分支执行。

![分支合并](/images/2022-03-28-git-branch-alias/merge-branch.svg)

## 四、命令集合

最后整理下所有的命令：

```sh
# 获取 feature Jira ID
fid() {
  local current_branch=$(git_current_branch)
  local arr=(`echo ${current_branch//\// }`)
  echo ${arr[@]:1:1}
}

# 创建开发分支并切换
alias gcbf='(){ git checkout -b feature/$1; }'
alias gcbh='(){ git checkout -b hotfix/$1; }'
alias gcbmt='(){ git checkout -b merge/$(fid)/test; }'
alias gcbmu='(){ git checkout -b merge/$(fid)/uat; }'
alias gcbmm='(){ git checkout -b merge/$(fid)/master; }'
alias gcbmr='(){ git checkout -b merge/$(fid)/release; }'

# git checkout master new branch
alias gcmnb='gcom && git checkout -b'
# git checkout master new feature
alias gcmnf='gcom && gcbf'
# git checkout release new hotfix
alias gcrnh='gcor && gcbh'

# 切换常备分支
alias gcot='git checkout test && git pull'
alias gcom='git checkout master && git pull'
alias gcou='git checkout uat && git pull'
alias gcor='git checkout release && git pull'

# 切换开发分支
alias gcof='(){ git checkout feature/$(fid); }'
alias gcoh='(){ git checkout hotfix/$(fid); }'
alias gcomt='(){ git checkout merge/$(fid)/test; }'
alias gcomu='(){git checkout merge/$(fid)/uat; }'
alias gcomm='(){ git checkout merge/$(fid)/master; }'
alias gcomr='(){ git checkout merge/$(fid)/release; }'

# 合并分支
alias gmf='(){ git merge feature/$(fid); }'
alias gmh='(){ git merge hotfix/$(fid); }'
alias gmt='git merge test'
alias gmu='git merge uat'
alias gmm='git merge master'
alias gmr='git merge release'

# 切换 merge 分支并合并 feature 更新
alias guft='gcomt && gmf'
alias gufu='gcomu && gmf'
alias gufm='gcomm && gmf'
alias gufr='gcomr && gmf'

# 切换 merge 分支并合并 hotfix 更新
alias guht='gcomt && gmh'
alias guhu='gcomu && gmh'
alias guhm='gcomm && gmh'
alias guhr='gcomr && gmh'

# 更新合并常备分支
gut() {
  local branch=$(git_current_branch)
  git checkout test
  git pull
  git checkout $branch
  git merge test
}

guu() {
  local branch=$(git_current_branch)
  git checkout uat
  git pull
  git checkout $branch
  git merge uat
}

gum() {
  local branch=$(git_current_branch)
  git checkout master
  git pull
  git checkout $branch
  git merge master
}

gur() {
  local branch=$(git_current_branch)
  git checkout release
  git pull
  git checkout $branch
  git merge release
}
```

## 参考链接

> - [Shell 函数 \| 菜鸟教程](https://www.runoob.com/linux/linux-shell-func.html)
> - [Git \- git\-config Documentation](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias)
> - [Git \- git\-symbolic\-ref Documentation](https://git-scm.com/docs/git-symbolic-ref)
> - [Shell 数组 \| 菜鸟教程](https://www.runoob.com/linux/linux-shell-array.html)
> - [macos \- zsh array from variable does not work \- Ask Different](https://apple.stackexchange.com/questions/351256/zsh-array-from-variable-does-not-work)
