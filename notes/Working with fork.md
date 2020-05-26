---
tags: [Git]
title: Working with fork
created: '2020-03-25T05:25:19.586Z'
modified: '2020-03-27T08:01:02.467Z'
---

# Working with fork

> A fork is a copy of a repository that you manage. Forks let you make **changes to a project without affecting the original repository**.
You can **fetch** updates from or **submit** changes to the **original repository** with **pull requests**.

GitHub 上的任何用户或组织都可对仓库复刻。 对仓库复刻类似于复制另一个仓库，主要有两点差异：
- You can use a pull request to suggest changes from your user-owned fork to the *original repository*, also known as the *upstream repository*.
- You can bring changes from the upstream repository to your local fork by synchronizing your fork with the upstream repository.

### 配置
> 必须在 Git 中配置指向上游仓库的远程仓库，才能将您在复刻中所做的更改同步 到原始仓库。 这也允许您将在原始仓库中所做的更改同步到复刻中。

1. Open Git Bash or Terminal.
2. List the current configured remote repository for you fork.
```bash
$ git remote -v
> origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
> origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```
3. Specify a new remote upstream repository that will be synced with the fork.
```bash
$ git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```
4. Verify the new upstream repository you've specified for your fork.
```bash
$ git remote -v
> origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
> origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
> upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
> upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```
