---
title: conda的工作流
categories: 工具/环境
---



## 虚拟环境

* 创建虚拟环境:

```bash
conda create -n 环境名 python=<python的版本>
```

* 查看所有的虚拟环境:

```bash
conda env list
```

* 切换虚拟环境:

```bash
conda activate <虚拟环境名>
conda deactivate
```



## Channel

* conda的channel就是conda的软件来源.

* conda默认会有一个channel叫`defaults`.

* 下载时默认指定去某个channel下载:

  ```bash
  conda install -c <channel>的名字 <软件包>
  ```

* conda-forge是社区维护的一个强大的软件源, 提供了更多的软件包, 并且频繁更新.

* conda换清华源: 

  * 首先生成`.condarc`文件:

    ```bash
    conda config --set show_channel_urls yes
    ```

  *  https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

  * 换好后, 使用`conda info`查看.
