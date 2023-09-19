<!-- # git漫游指南 -->

### 创建新分支同时切换到新分支

```Vue
git checkout -b <your-branch-name>
其中"-b"是branch的缩写

我们也可以：
git checkout -b <your-branch-name> <origin/远程分支名>
在它的基础上 创建新分支
```

### git branch的使用

```Vue
1 列出分支 git branch (-r 查看远程分支 -a 查看本地和远程分支)
2 创建新分支 git branch <branch-name>
3 删除分支 git branch -d <branch-name> (如果分支上的更改还没有合并到其他分支，git会阻止删除分支)
4 重命名 git branch -m <new-branch-name>
5 强制移动 git branch -f main HEAD~3 (将本地分支main移动到当前分支的前三个提交之前的位置 如果有未提交的更改，那么这些更改会随着移动分支后丢失，建议用git stash保存)
```

### 分支合并

* git merge

```Vue
使用该git合并两个分支会产生一条特殊记录
（Merge branch 'xxxx' of xxxx into xxxx）
```

* git rebase

```Vue
使用该git操作 类似于取出一系列的提交记录，复制它们，然后在另一个地方逐个放下去，不会产生特殊记录
```

### 在提交树上移动

```Vue
一、分离HEAD
    让其指向了某个具体的提交记录，而不是分支名
    git checkout <提交记录哈希值>
    注：分离HEAD可以让你在不同的分支之间进行切换，甚至可以切换到不同分支的特定提交记录上
    实例：
        HEAD -> main -> c1
        git checkout c1
        HEAD -> c1
```

### 查看HEAD指向

```Vue
git rev-parse HEAD 返回当前HEAD所在提交记录哈希值
git show HEAD 查看HEAD指向的提交详细信息
```

### 撤销变更

* git reset

```Vue
git reset HEAD~1
在reset后，所做的变更还在，但是处于未加入暂存区状态
```

* git revert

```Vue
revert操作并不会删除提交记录 而是重新创建一个提交 提交的内容是要撤销内容回滚
```

### 整理提交记录

* git cherry-pick

```Vue
git cherry-pick <提交号>
如果你想将一些提交复制到当前所在的位置（HEAD）下面的话，cherry-pick最简单直接

转移多次提交
git cherry-pick <HashA> <HashB>
如果你想要一堆提交转移 可以使用：
git cherry-pick <HashA>..<HashB>
上面的命令要保证提交A早于提交B，否则命令将失败，但不会报错；注意，提交A不会在命令当中，如果想要包含 需要使用下面的语法
git cherry-pick <HashA>^..<HashB>
```

* 交互式rebase

```Vue
交互式rebase指的是使用带参数的（--interactive）的rebase命令，即-i

当rebase UI界面打开时 你能做三件事：
1. 调整提交记录的顺序（通过鼠标拖放来完成）
2. 删除你不想要的提交（通过切换pick的状态来完成，关闭就意味着你不想要这个提交记录）
3. 合并提交 允许你把多个提交记录合并成一个

squash(s) 可以将本次提交压缩到上方的提交中，git是你有机会编写更改后的提交信息
fixup(f) 和squash一样 但是不能修改f的提交记录的提交信息
```

### 远程主机名（git remote）

```Vue
git remote(列出所有远程主机)
在我们克隆版本库的时候，所使用的远程主机名自动被Git命名为origin
git remote <原主机名> <新主机名> 修改远程主机名 改完就不是 git push origin 了哦
```

### 追踪远程分支 （git branch --set-upstream <本地分支名> <远程分支名>）

```Vue
Git会自动在本地分支和远程分支之间 建立一种追踪关系 master 会 自动追踪"origin/master"分支

Git也允许手动建立追踪关系
git branch --set-upstream master origin/next
上面命令指定master 追踪origin/next 分支

如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名
git pull origin
上面命令表示 本地当前分支与对应的origin追踪分支进行合并
如果当前分支只有一个追踪分支，连远程主机名都可以省略
git pull
```

### 删除远程/本地分支

```Vue
删除远程分支：
    git push origin :<要删除的远程分支名> (这个相当于将空分支上传到远程)
    git push origin --delete <要删除的远程分支名>
删除本地分支:
    git branch -d <要删除的本地分支名>
    要删除的分支的提交记录未合并到其他分支上：
        git branch -D <要删除的本地分支名> (强制删除)
```
