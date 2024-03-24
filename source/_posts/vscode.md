---
title: 使用vscode作为更好的preview IDE
categories: 工具/环境
---



## C/C++ LSP

首先可以使用CMake或者Bear生成整个项目的`compile_commands.json`.

语法解析可以使用`clangd`插件.

对于具体项目, 可以在项目中新建`.vscode/settings.json`.

然后在其中添加:

```json
{
    "clangd.arguments": [
        "--compile-commands-dir=这里就是你的compile_commands.json的路径"
    ]
}
```



## Task Manager

Task Manager可以让你预定义一些与项目有关的任务.

首先在项目中新建`.vscode/tasks.json`.

