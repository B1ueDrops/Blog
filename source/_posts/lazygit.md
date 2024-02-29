---
title: lazygit的一些常见用法
categories: 小工具
---

> 在一个pane中使用`?`可以查看cheetsheet.



## 正确的回滚操作

* 第一: 基于当前分支新建分支(branch pane中使用`n`).
* 第二: 切换到新建分支 (branch pane中使用空格).
* 第三: 回滚到特定的commit (commit pane中选中特定的commit之后, 用`g`, 基于`git reset`进行修改).
* 第四: 选择暂存区中需要保留或舍弃的修改(file pane中使用`d`或者空格).
* 第五: 将暂存区中的修改进行commit之后, 就可以在回滚分支进行开发.



## Interactive Rebasing

在commit pane中, 对于某次commit使用`e`后, 这次commit及其上面的所有commit都会进入interactive-rebasing的状态.

每个commit都会有四种状态:

* `pick`: 保留这次commit.
* `squash`: 和下面的commit合并, 其中commit的信息也会被合并.
* `drop`: 删除此次commit.
* `fixup`: 和下面的commit合并, 但是这次commit的信息会消失.

对于每个commit设置完状态之后, 使用`m`, 选择`continue`, 就可以生效.



## 改写commit信息

在commit pane中, 使用`r`就可以改写commit信息.

