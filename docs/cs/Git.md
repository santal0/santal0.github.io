# Github 速查


## 1. 创建仓库

## 2. 克隆仓库

### 2.1 fork
### 2.2 clone
```
git clone <url>

```

## 3. 同步仓库
添加上游仓库
```bash
git remote add upstream <url>

git remote -v  #检查
```

https://blog.csdn.net/inthat/article/details/124779375

从上游仓库fetch最新更改
```bash
git fetch upstream
```
更新本地分支以匹配上游仓库假设我们想保持本地 main 分支与上游仓库的 main 分支完全一致，可以执行以下命令：

```bash
git checkout main 
git reset --hard upstream/main # 不一定是main分支，可以替换为其他分支
```
此命令会先切换到本地 main分支，覆盖本地文件至本地 main分支的版本。之后，强制将本地 main 分支更新为与上游仓库的 main 分支完全一致，包括文件和commit history。使用命令时，任何未提交的更改都会丢失。因此，在执行此命令之前，确保所有未提交的工作已经保存或提交。

将更新后的分支Push到 GitHub 中的 Fork 仓库
```bash
git push origin main --force
```

通过此操作，你的 fork 仓库中的 main 分支将与上游仓库保持完全同步。实际操作中，也可以用main以外的branch进行同步。

转载自：[Github操作：Fork仓库与上游仓库的同步 - 乔拾壹的文章 - 知乎](https://zhuanlan.zhihu.com/p/715381563)

从 fork 后的仓库抓取新的分支到本地

```bash
git checkout -b feat-lab1-3-DiskManager origin/feat-lab1-3-DiskManager
```



## 4. 分支指令

查看所有分支
```bash
git branch
```

创建分支
```bash
git branch <branch-name>
```

切换分支
```bash
git checkout <branch-name>
```

创建并切换分支
```bash
git checkout -b <branch-name>
```

合并分支：
先切换回主分支，然后执行
```bash
git merge <branch-name>
# 这里的 branch-name 是要被合并的分支名
```

删除已经合并的分支
```bash
git branch -d <branch-name>
```

