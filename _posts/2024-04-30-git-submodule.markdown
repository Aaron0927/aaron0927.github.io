---
layout: post
title:  "Git Submodule"
date:   2024-04-30 23:38:00 +0800
categories: jekyll blog
---


用于将一个 git 仓库作为另一个仓库的子模块进行管理，并保持独立的仓库结构
1. 主仓库不会直接修改子模块仓库的代码或文件。子模块仓库是独立的，它的代码和提交历史由子模块仓库自身管理。
2. 主仓库只记录子模块的仓库地址和特定提交，以及子模块在主仓库中的存放位置。
3. 当你对子模块进行操作时，你需要在子模块仓库的目录下使用 git 命令，例如git pull更新子模块

如果把当前仓库当成主仓库，并添加子模块时，并不会影响该仓库的历史记录和代码。主仓库只会记录子模块的引用，而不会修改主仓库的历史。

**添加子模块**

1. 添加子模块：在 git 仓库中，使用以下命令添加一个子模块：
    ```shell
    git submodule add <repository_URL> <path_to_submodule_directory>
    ```
> 这将在当前仓库中添加一个子模块，其中<repository_URL>是子模块的 git 仓库地址，<path_to_submodule_directory>是子模块在当前仓库中的存放路径。

    添加子模块时可以指定分支：
    ```shell
    git submodule add -b <branch_name> <repository_URL> <path_to_submodule_directory>
    ```
2. 提交更改：当所有子模块添加完成后，提交主仓库的更改


**克隆含有子模块的仓库**

可以使用以下命令：
```shell
git clone <repository_URL>
git submodule init
git submodule update
```
这将更新到本地分支的最新提交

使用`git submodule update --remote`,将更新到远程仓库的最新提交

**更新子模块**

1. 更新子模块内容：在主仓库中更新子模块的内容，可以使用以下命令：
    ```shell
    git submodule update --remote
    ```
2. 提交更改：提交子模块的更新以及主仓库的更改

**删除子模块**

1. 删除子模块引用：可以通过以下步骤删除子模块：
    ```shell
    git submodule deinit -f -- <path_to_submodule_directory>
    rm -rf .git/modules/<path_to_submodule_directory>
    git rm -f <path_to_submodule_directory>
    ```
2. 提交更改：提交主仓库的更改

**管理子模块**

* 使用 `git submodule status` 查看当前子模块的状态。
* 在克隆或更新仓库时，使用 `git submodule update` 更新子模块内容。
* 若要在主项目和子模块中都完成操作，需要分别进入主项目目录和子模块目录进行操作。

子模块的分支和仓库存放路径在`.gitmodules`中，可以在主仓库中新建分支，并修改其中的内容，就可以管理多个版本了

更新主仓库的.gitmodules文件：
```shell
git submodule sync
```