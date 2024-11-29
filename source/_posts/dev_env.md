---
title: 快速配置完备开发环境(本地/服务器)
categories: 开发工具
---



## Installation

> 本地

* Homebrew: https://docs.brew.sh/Installation

* 然后下面是Homebrew的下载配置: 假设文件名称叫`Brewfile`

  ```bash
  brew "eza"
  brew "bat"
  brew "yazi"
  brew "zellij"
  brew "zoxide"
  brew "neovim"
  brew "ffmpeg"
  brew "ripgrep"
  brew "lazygit"
  brew "wezterm"
  brew "fastfetch"
  brew "hammerspoon"
  brew "ffmpegthumbnailer"
  ```

* Typora下载: https://typora.io/#feature

* vscode下载: https://code.visualstudio.com/download

  * 下载CLI版本, 二进制文件名称叫`code`.
  * 放入`~/bin`中.




## Config



### Shell启动脚本

> 本地

* 位置是`~/.zshrc`

```bash
# All ALIAS
alias ra="yazi"
alias nvim="lvim"
alias lg="lazygit"
alias ls="eza --icons"
alias la="eza --icons -a"
alias lt="eza --icons -T"
alias ll="eza --icons -l"
alias ty="open -a typora"

# ALL ENV VAR
export EDITOR='lvim'
export PATH=$HOME'/bin':${PATH}

# All Util Functions
hide_icon() {
  defaults write com.apple.finder CreateDesktop -bool FALSE; killall Finder
}
show_icon() {
  defaults write com.apple.finder CreateDesktop -bool true; killall Finder
}

# Start plugins
bindkey -v
bindkey "^H" backward-delete-char
bindkey "^?" backward-delete-char
bindkey -M vicmd "B" vi-beginning-of-line
bindkey -M vicmd "W" vi-end-of-line
bindkey -M vicmd "H" vi-forward-word
bindkey -M vicmd "L" vi-forward-word
function zle-keymap-select {
	if [[ ${KEYMAP} == vicmd ]] || [[ $1 = 'block' ]]; then
		echo -ne '\e[1 q'
	elif [[ ${KEYMAP} == main ]] || [[ ${KEYMAP} == viins ]] || [[ ${KEYMAP} = '' ]] || [[ $1 = 'beam' ]]; then
		echo -ne '\e[5 q'
	fi
}
zle -N zle-keymap-select
```



### vscode

> 本地

* Task配置: `.vscode/tasks.json`

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



### vim/neovim

> 本地

* 可以选择LunarVim作为发行版: https://www.lunarvim.org/docs/installation
  * 下载之后, 自定制的部分在: `~/.config/lvim/config.lua`

```lua
-- LVIM Options
vim.o.expandtab = false
vim.o.tabstop = 4
vim.o.shiftwidth = 4
vim.o.softtabstop = 4

-- LVIM Mapping
lvim.keys.normal_mode['H'] = "5h"
lvim.keys.normal_mode['J'] = "5j"
lvim.lsp.buffer_mappings.normal_mode['K'] = nil
lvim.keys.normal_mode['K'] = "5k"
lvim.keys.normal_mode['L'] = "5l"
lvim.keys.normal_mode['B'] = "^"
lvim.keys.normal_mode['W'] = "$"
lvim.keys.visual_mode['H'] = "5h"
lvim.keys.visual_mode['J'] = "5j"
lvim.keys.visual_mode['K'] = "5k"
lvim.keys.visual_mode['L'] = "5l"
lvim.keys.visual_mode['B'] = "^"
lvim.keys.visual_mode['W'] = "$"

-- LVIM Plugins
lvim.plugins = {
	{
		"kylechui/nvim-surround",
		version = "*", -- Use for stability; omit to use `main` branch for the latest features
		event = "VeryLazy",
		config = function()
			require("nvim-surround").setup({
				-- Configuration here, or leave empty to use defaults
			})
		end
	}
}
```



> 服务器






## Usage



### yazi

* 常用键位:

| 键位 | 功能                            |
| ---- | ------------------------------- |
| `z`  | 用zoxide跳转目录                |
| `/`  | 查找目录                        |
| `o`  | 打开文件                        |
| `g`  | 跳转到home, downloads等常见目录 |
| `r`  | 重命名文件                      |
| `O`  | Open Interactive                |



### zellij

* 常用键位:

| 键位       | 功能               |
| ---------- | ------------------ |
| `Alt+n`    | 新建Pane           |
| `Alt+hjkl` | 切换Pane Focus/Tab |
| `Alt+f`    | 新建Float Pane     |

