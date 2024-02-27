---
title: Neovim/Nvchad键位配置
categories: 小工具
---


## NvChad使用手册

> 查看Cheetsheet可以使用`<LEADER>ch`.

1. 当你在很多很多作用域嵌套的时候, 如果你想到一个作用域的最外面, 可以使用`<LEADER>cc`.

2. 快速注释和解注释是`<LEADER>/`.

3. 关于行号:
   * Toggle行号是用`<LEADER>n`.
   * Toggle相对行号是用`<LEADER>rn`.
   * 相对行号可以快速查看一个代码块有多少行.

4. 格式化:

   * 可以使用`<LEADER>fm`进行全局格式化.

5. 关于终端:

   * NvChad中终端的相关快捷键使用了Alt, 不同终端对于Alt键进行了不同的处理, <font color=red>iTerm2中默认禁用了终端</font>, 如果要开启终端, 需要如下设置:

     ![iTerm2关于alt键的设置](iterm2-alt.png)

   * Toggle水平终端: `Alt+h`.
   * Toggle垂直终端: `Alt+v`.
   * Toggle浮动终端: `Alt+i`.

6. 关于Git:
   * 查看当前文件中的修改(NvChad中叫hunk), 可以使用`[c`和`]c`.
   * 在一处修改(尤其是删除)上, 使用`<LEADER>ph`, 可以对修改进行preview, 清楚的看到改动了什么.
   * 对任意文件的任意一行, 如果你想看这一行的commit信息, 可以使用`<LEADER>gb`, <font color=red>十分强大</font>.
   * 如果你做了删除操作, 但是你想Toggle一下, 看看自己删除了什么, 可以使用`<LEADER>td`, 或者你想撤销一下, 可以用`<LEADER>rh`.
   * 用`<LEADER>cm`可以通过Telescope查看所有的commit.
   * 用`<LEADER>gt`可以用Telescope查看所有git status.

7. 关于文件树:
   * `<LEADER>e`是focus文件树.
   * `<C-n>`是Toggle文件树.

8. 关于Tab:

   * 可以使用`<Tab>`和`<S-Tab>`切换不同的Tab.

9. 关于搜索:

   * `<ESC>`用来代替`:nohl`.
   * 用`<LEADER>fw`可以使用ripgrep查找内容.
   * 用`<LEADER>ff`可以用ripgrep模糊查找文件.
