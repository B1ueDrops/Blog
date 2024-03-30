---
title: yabai窗口管理器键位配置
categories: 工具/环境
---



## 关闭SIP的方法

1. 首先, 进入Mac的Recovery Mode.
2. 进入终端: [实用工具]-[终端].
3. 终端中输入: `csrutil disable`.
4. 重启之后, 输入`csrutil disable`, 查看SIP是否被disable.
5. 之后, 在终端运行: `sudo nvram boot-args=-arm64e_preview_abi`.
6. 然后重启.



## 下载完yabai之后

在`/etc/sudoers.d`中, 创建`yabai`, 写入:

```
<user> ALL=(root) NOPASSWD: sha256:<hash> <yabai> --load-sa
```

* `user`: `whoami`的输出.
* `hash`: `shasum -a 256 $(which yabai)`的输出.
* `yabai`: `which yabai`的输出.

<font color=red>注意, yabai更新之后, hash的值会改变, 需要重新修改.</font>

## yabai的键位配置

| 功能                         | 键位                     |
| ---------------------------- | ------------------------ |
| 切换窗口聚焦                 | `cmd+alt+h,j,k,l`        |
| 旋转窗口布局90度             | `cmd+alt+;`              |
| 最大化窗口                   | `cmd+alt+m`              |
| 关闭窗口                     | `cmd+alt+c`              |
| 移动窗口位置                 | `ctrl+cmd+alt+h,j,k,l`   |
| 切换虚拟桌面                 | `cmd+alt+[]`             |
| 创建虚拟桌面                 | `ctrl+cmd+alt+n`         |
| 销毁虚拟桌面                 | `ctrl+cmd+alt+c`         |
| 切换到某个虚拟桌面           | `cmd+alt+1/2/3/4/5`      |
| 将当前窗口移动到某个虚拟桌面 | `ctrl+cmd+alt+1/2/3/4/5` |
| 重启yabai                    | `ctrl+cmd+alt+r`         |
| 将窗口布局改为浮动           | `ctrl+cmd+alt+f`         |
| 隐藏某个窗口                 | `cmd+h`                  |
| 打开某个隐藏的窗口           | `cmd+tab`                |



## sketchybar下载依赖

* sketchybar-app-font: https://b1uedrops.github.io/static/files/sketchybar-app-font.ttf

