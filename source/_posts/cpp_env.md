---
title: C语言系列环境工作区配置
categories: 开发工具
mathjax: true
---

>下面的所有安装基本假设在远程环境.



## 工具下载

* 首先下载一些基本的工具, 然后配置到环境变量中.

  * gdb: 服务器上应该有.
  
  * cmake: 服务器上应该也有.
  
  * clangd: https://github.com/clangd/clangd/
  
    * 注意, 下载`clangd`之后, 不要把二进制文件拷贝到别处, 因为需要链接本地文件夹的库文件.
  
  * clang-tidy:
  
    ```bash
    pip install clang-tidy
    ```
  
    
  

## 插件配置

* 所有插件列表如下: `.vscode/extensions.json`

  ```txt
  {
  	"recommendations": [
  		"twxs.cmake",
  		"llvm-vs-code-extensions.vscode-clangd",
  	]
  }
  ```

  * 然后在`marketplace`用`@recommand`搜索一下即可完全下载.

* 首先需要生成`compile_commands.json`:

  * 如果是`cmake`, 那么在原有的`cmake`命令之后, 需要加上`-DCMAKE_EXPORT_COMPILE_COMMANDS=1`.

  * 如果是`make`:

    * 首先下载工具:

      ```bash
      pip install scan-build
      ```

    * 然后生成`compile_commands.json`:

      ```bash
      intercept-build <make命令>
      ```

* 然后在`.vscode/settings.json`中加入如下配置, 具体配置根据工程设置:

  ```json
  {
  	"editor.insertSpaces": false,
  	"editor.detectIndentation": false,
  	"cmake.cmakePath": "cmake",
  	"clangd.path": "/home/ligaog/clangd_18.1.3/bin/clangd",
  	"clangd.arguments": [
  		"--compile-commands-dir=${workspaceFolder}/"
  	]
  }
  ```

* 然后配置`.vscode/tasks.json`, 注意编译的时候要加上`-g`选项方便调试.

  ```json
  {
  	"version": "2.0.0",
  	"tasks": [
  		{
  			"label": "Name of your task",
  			"type": "shell",
  			"command": "command_name",
  			"args": [ "-pql", "MoonVisual.py", "MoonVisual" ],
  			"options": { "cwd": "${workspaceFolder}/src" }
  		}
  	]
  }
  ```

  * 有时候一个Task由很多个命令组成, 那么`tasks.json`参考, 通过`dependsOn`字段将任务连成拓扑序.

    ```json
    {
    	"version": "2.0.0",
    	"tasks": [
    		{
    			"label": "task-1",
    			"type": "shell",
    			"command": "echo 'Running command 1'",
    			"options": {
    				"cwd": "${workspaceFolder}"
    			}
    		},
    		{
    			"label": "task-2",
    			"type": "shell",
    			"command": "echo 'Running command 2'",
    			"options": {
    				"cwd": "${workspaceFolder}/.vscode"
    			},
    			"dependsOn": ["task-1"]
    		},
    		{
    			"label": "final-task",
    			"type": "shell",
    			"command": "echo 'Final Task'",
    			"dependsOn": ["task-2"]
    		}
      ]
    }
    ```

* 然后配置`.vscode/launch.json`, 其中的`preLaunchTask`是在之前Task中任务的`label`, 然后`ctrl+x`可以开启调试

  ```json
  {
  	"version": "0.2.0",
  	"configurations": [
  		{
  			"name": "Debug C/C++ Code",
  			"type": "cppdbg",
  			"request": "launch",
  			"program": "${workspaceFolder}/<path-to-binary>",
  			"args": [],
  			"stopAtEntry": false,
  			"cwd": "${workspaceFolder}",
  			"environment": [],
  			"externalConsole": false,
  			"MIMode": "gdb",
  			"setupCommands": [
  				{
  				"description": "Enable pretty-printing",
  				"text": "-enable-pretty-printing",
  				"ignoreFailures": true
  				}
  			],
  			"miDebuggerPath": "/usr/bin/gdb", 
  			"preLaunchTask": "build",
  			"logging": {
  				"engineLogging": false
  			}
  		}
    ]
  }

