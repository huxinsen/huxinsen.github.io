---
layout: post
title: Git cheatsheet
description: "often used cmds of Git"
share: false
tags: [Git]
image:
  feature: 2020-03-02-git-cheatsheet/branching-illustration.png
  credit: git
  creditlink: https://git-scm.com/images/branching-illustration@2x.png
---

## 分支重命名

---

1. 本地分支重命名

   ```
   git branch -m oldName  newName
   ```

2. 将重命名后的分支推送到远程

   ```
   git push origin newName
   ```

3. 删除远程的旧分支

   ```
   git push --delete origin oldName
   ```

## fork 项目

---

先 fork 一个项目

### 一、同步远程仓库更新

1. 查看目前仓库可以远程更新的信息

   ```
   git remote -v
   ```

2. 配置一个远程更新链接

   ```
   git remote add upstream git@github.com:xxx/xxx.git
   ```

3. 拉取远程仓库的代码

   ```
   git fetch upstream
   ```

4. 切换到想要 merge 的分支，这里用 master

   ```
   git checkout master
   ```

5. 合并远程仓库的代码

   ```
   git merge upstream/master
   ```

6. 把远程仓库的代码作为新源提交到自己的服务器仓库中

   ```
   git push
   ```

   注：3、4、5 合并命令：

   ```
   git pull upstream master
   ```

### 二、远程仓库有新分支

1. 在本地新建一个分支，该分支的名称最好与源项目中新增的那个分支的名称相同以便区分

   ```
   git checkout -b 新分支名称
   ```

2. 从源项目中将新分支的内容 pull 到本地

   ```
   git pull upstream 新分支名称
   ```

3. 两步合并：

   ```
   git checkout -b 新分支名称 upstream/新分支名称
   ```

4. 将 pull 下来的分支 push 到自己的项目中去

   ```
   git push origin 新分支名称
   ```

## 合并多个 commit

---

1. 查看提交历史，git log

   假如最近 4 条历史如下：

   ```
   commit 3ca6ec340edc66df13423f36f52919dfa3......

   commit 1b4056686d1b494a5c86757f9eaed844......

   commit 53f244ac8730d33b353bee3b24210b07......

   commit 3a4226b4a0b6fa68783b07f1cee7b688.......
   ```

   历史记录是按照时间排序的，时间近的排在前面。

2. git rebase

   想要合并 1-3 条，有两个方法：

   1. 从 HEAD 版本开始往过去数 3 个版本

      ```
      git rebase -i HEAD~3
      ```

   2. 指名要合并的版本之前的版本号

      ```
      git rebase -i 3a4226b
      ```

      请注意 3a4226b 这个版本是不参与合并的，可以把它当做一个坐标

3. 选取要合并的提交，根据提示操作

## 发布项目到 gh-pages 分支

---

1. 进入 build 文件夹下

   ```
   cd build
   ```

2. git 初始化

   ```
   git init
   ```

3. 创建 gh-pages 分支

   ```
   git checkout --orphan gh-pages
   ```

4. 添加文件到暂存区

   ```
   git add .
   ```

5. 添加提交信息

   ```
   git commit -m "init project"
   ```

6. 设置远程仓库地址

   ```
   git remote add origin git@github.com:xxx/xxx.git
   ```

7. 推送项目到 gh-pages 分支

   ```
   git push origin gh-pages
   ```

## 合并 commit 的简便方法

---

先撤销过去 5 个 commit，然后再建一个新的。

```
$ git reset HEAD~5
$ git add .
$ git commit -am "Here's the bug fix that closes #28"
$ git push --force
```

## 回退 git merge

---

1. git checkout 到要恢复的那个分支上

   ```
   git checkout develop
   ```

2. git reflog 查出要回退到 merge 前的版本号

   ```
   git reflog
   ```

3. git reset --hard [版本号]就回退到 merge 前的代码状态了

   ```
   git reset --hard 22d9f21
   ```

## git clone 慢

---

1. clone 最新的提交

   ```
   git clone --depth=1 git@github.com:xxx/xxx.git
   ```

2. 更新获取完整历史版本
   ```
   git fetch --unshallow
   ```
