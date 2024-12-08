---
title: 快速配置基础的开发环境
categories: 开发工具
---



在工作中, 一般会有两种形式的环境:

* 本地环境: 一般会是一个Windows电脑, 和一个Ubuntu电脑, 两种环境都会有管理员权限.
* 远程环境: 
  * 可能是一个Docker容器, 可能在本地, 也可能在远程.
  * 可能是一个Ubuntu, 但是没有管理员权限, 甚至可能没有联网, 但一般会有资源.


下面会介绍如何定制不同环境, 来配置一个比较舒服, 并且稳定的开发环境.



## Ubuntu本地环境

> 在这个环境下你有`sudo`权限.

### 软件源和必要的软件

* 首先查看一下你的指令集架构, 以及Linux发行版的版本:

  ```bash
  uname -m && cat /etc/*release
  ```

* 配置`apt`的清华源: 根据你的指令集架构和Linux发行版选择对应的软件源

  ```bash
  # x86架构
  firefox https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/
  # 非x86架构
  firefox https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports/
  # 备份/etc/apt/sources.list
  sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
  # 打开sources.list, 写入镜像源
  sudo gedit /etc/apt/sources.list
  # 更新软件源
  sudo apt update
  # 下载必要的包
  sudo apt install -y build-essential neovim git curl neofetch zsh fzf zoxide
  ```

* 安装lazygit

  ```bash
  firefox https://github.com/jesseduffield/lazygit?tab=readme-ov-file#ubuntu
  ```
  
* 下载Rust工具链:

  ```bash
  firefox https://www.rust-lang.org/tools/install
  # 下载Rust的一些工具
  cargo install eza bat yazi-fm yazi-cli
  ```
  
* 安装NerdFont终端字体:

  ```bash
  # nerdfont官网选择喜欢的字体, 终端字体可选0xProto Nerd Font, 然后复制链接地址
  firefox https://www.nerdfonts.com/font-downloads
  # 创建用户字体目录
  mkdir ~/.fonts
  # 下载字体(链接是从上面的网站复制的)
  wget -P ~/.fonts https://github.com/ryanoasis/nerd-fonts/releases/download/v3.3.0/0xProto.zip
  # 解压字体
  cd ~/.fonts && unzip 0xProto.zip
  # 安装字体
  fc-cache -fv

* 安装Consolas编程字体:
  
  ```bash
  firefox https://github.com/yakumioto/YaHei-Consolas-Hybrid-1.12
  ```
  
* 然后安装终端模拟器WezTerm: https://wezfurlong.org/wezterm/install/linux.html
  
  * 之后, WezTerm的配置文件为: `~/.config/wezterm/wezterm.lua`
  
    ```lua
    local wezterm = require('wezterm')
    local act = wezterm.action
    
    local config = {
    	font = wezterm.font {
    		family = '0xProto Nerd Font',
    	},
    	font_size = 20.0,
    	line_height = 1.4,
    	color_scheme = 'tokyonight',
    	enable_tab_bar = false,
    	default_cursor_style = 'SteadyBar',
      -- Mouse right click to copy and paste
    	mouse_bindings = {
    		{
    			event = { Down = { streak = 1, button = "Right" } },
    			mods = "NONE",
    			action = wezterm.action_callback(function(window, pane)
    				local has_selection = window:get_selection_text_for_pane(pane) ~= ""
    				if has_selection then
    					window:perform_action(act.CopyTo("ClipboardAndPrimarySelection"), pane)
    					window:perform_action(act.ClearSelection, pane)
    				else
    					window:perform_action(act({ PasteFrom = "Clipboard" }), pane)
    				end
    			end),
    		},
    	}
    }
    return config
    ```

* 安装Docker:

  ```bash
  firefox https://docs.docker.com/engine/install/ubuntu/
  ```

  * Docker移除`sudo`:

    ```bash
    sudo groupadd docker
    sudo gpasswd -a $USER docker
    docker run hello-world
    # 然后重新logout, 然后login
    ```


### Shell

* 然后配置Shell:

  ```bash
  # 修改默认终端是zsh
  chsh -s $(which zsh)
  # 下载oh-my-zsh
  firefox https://ohmyz.sh/#install
  ```

  * 下载必要的zsh插件:

  ```bash
  git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
  ```

  * 还有一个插件叫`powerlevel10k`:

  ```bash
  firefox https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#oh-my-zsh
  ```

  * 然后在`~/.zshrc`中注册插件:

  ```bash
  plugins=(git zoxide zsh-autosuggestions zsh-syntax-highlighting)
  ZSH_THEME="powerlevel10k/powerlevel10k"
  ```

  * 下面是通用的zsh的配置文件: `~/zsh_user`, 需要在`.zshrc`最后`source $HOME/zsh_user`

  ```bash
  alias ra='yazi'
  alias cls='clear'
  alias lg='lazygit'
  alias ls='eza --icons'
  alias la='eza --icons -a'
  alias ll='eza --icons -l'
  alias gd='cd ~/Downloads'
  
  if [[ -n $SSH_CONNECTION ]]; then
    export EDITOR='vim'
  else
  	export EDITOR='nvim'
  fi
  
  bindkey -v
  bindkey '^[q' autosuggest-accept
  function move-five-right {
  	for ((i=0; i<5; i++)); do
      		zle vi-forward-char 
  	done
  }
  function move-five-left {
  	for ((i=0; i<5; i++)); do
      		zle vi-backward-char 
  	done
  }
  function zle-keymap-select {
  	if [[ ${KEYMAP} == vicmd ]] || [[ $1 = 'block' ]]; then
  		echo -ne '\e[1 q'
  	elif [[ ${KEYMAP} == main ]] || [[ ${KEYMAP} == viins ]] || [[ ${KEYMAP} = '' ]] || [[ $1 = 'beam' ]]; then
  		echo -ne '\e[5 q'
  	fi
  }
  zle -N move-five-left
  zle -N move-five-right
  zle -N zle-keymap-select
  bindkey "^H" backward-delete-char
  bindkey "^?" backward-delete-char
  bindkey -M vicmd "B" vi-beginning-of-line
  bindkey -M vicmd "W" vi-end-of-line
  bindkey -M vicmd 'H' move-five-left
  bindkey -M vicmd 'L' move-five-right
  ```



### vscode

* 然后下载`vscode`的`deb`:

  ```bash
  firefox https://code.visualstudio.com/Download
  ```

  * 所有需要下载的`vscode`插件列表`id`在`extensions-list.txt`:

    ```txt
    vscodevim.vim
    usernamehw.errorlens
    letrieu.expand-region
    naumovs.color-highlight
    PKief.material-icon-theme
    ms-vscode.remote-explorer
    jeff-hykin.macro-commander
    alefragnani.project-manager
    ms-vscode-remote.remote-ssh
    74th.monokai-charcoal-high-contrast
    ```

  * 只需要用`cat extensions-list.txt | xargs -L 1 code --install-extension`就可以完全下载.

  * 用户级别的`settings.json`在`~/.config/Code/Users/settings.json`:

    ```json
    {
      "workbench.colorTheme": "monokai-charcoal",
    	"editor.fontSize": 20,
      "editor.lineHeight": 1.5,
      "editor.wordWrapColumn": 30,
      "editor.fontFamily": "YaHei Consolas Hybrid",
      "editor.guides.bracketPairs": "active",
      "editor.stickyScroll.enabled": true,
      "editor.insertSpaces": false,
      "editor.detectIndentation": false,
      "editor.renderWhitespace": "boundary",
      "editor.linkedEditing": true,
      "editor.suggest.insertMode": "replace",
    	"editor.minimap.enabled": false,
    	"editor.suggest.snippetsPreventQuickSuggestions": false,
    	"editor.suggestSelection": "recentlyUsed",
    	"editor.suggest.preview": true,
    	"editor.acceptSuggestionOnEnter": "smart",
    	"editor.cursorBlinking": "phase",
    	"editor.cursorSmoothCaretAnimation": "on",
    	"workbench.startupEditor": "none",
    	"workbench.iconTheme": "material-icon-theme",
    	"workbench.layoutControl.enabled": false,
    	"workbench.activityBar.location": "default",
    	"window.commandCenter": false,
    	"window.zoomLevel": 1.1,
    	"terminal.integrated.fontSize": 18,
    	"terminal.integrated.defaultLocation": "view",
    	"terminal.integrated.fontFamily": "'0xProto Nerd Font'",
    	"terminal.integrated.lineHeight": 1.4,
    	"terminal.integrated.commandsToSkipShell": [
    		"workbench.view.explorer",
    		"workbench.view.extensions",
    		"workbench.action.toggleZenMode",
    		"workbench.action.focusLeftGroup",
    		"workbench.action.focusRightGroup",
    		"workbench.action.focusBelowGroup",
    		"workbench.action.focusAboveGroup",
    		"workbench.action.closeActiveEditor",
    		"workbench.action.toggleEditorWidths",
    	],
    	"zenMode.hideLineNumbers": false,
    	"zenMode.centerLayout": false,
    	"zenMode.showTabs": "none",
    	"zenMode.fullScreen": false,
    	"files.autoSave": "afterDelay",
    	"files.autoSaveDelay": 500,
    	"vim.incsearch": true,
    	"vim.smartRelativeLine": true,
    	"vim.useSystemClipboard": false,
    	"vim.hlsearch": true,
    	"vim.leader": "<space>",
    	"vim.foldfix": true,
    	"vim.cursorStylePerMode.visual": "block-outline",
    	"vim.highlightedyank.enable": true,
    	"vim.normalModeKeyBindings": [
    		{ "before": ["J"], "after": ["6", "j"] },
    		{ "before": ["K"], "after": ["6", "k"] },
    		{ "before": ["H"], "after": ["6", "h"] },
    		{ "before": ["L"], "after": ["6", "l"] },
    		{ "before": ["W"], "after": ["$"] },
    		{ "before": ["B"], "after": ["^"] },
    		{ "before": ["<leader>", "n"], "commands": [":nohl"] },
    		{ "before": ["<leader>", "b"], "commands": ["editor.debug.action.toggleBreakpoint"] },
    		{ "before": ["<leader>", "i"], "commands": ["editor.emmet.action.incrementNumberByOne"] },
    		{ "before": ["enter"], "commands": ["expand_region"] },
    		{ "before": ["g", "d"], "commands": ["editor.action.revealDefinitionAside"] },
    		{ "before": ["g", "h"], "commands": ["editor.action.showHover"] },
    		{ "before": ["g", "r"], "commands": ["editor.action.goToReferences"] },
        ],
    	"vim.visualModeKeyBindings": [
    		{ "before": ["K"], "after": ["6", "k"] },
    		{ "before": ["H"], "after": ["6", "h"] },
    		{ "before": ["J"], "after": ["6", "j"] },
    		{ "before": ["L"], "after": ["6", "l"] },
    		{ "before": ["W"], "after": ["$"] },
    		{ "before": ["B"], "after": ["^"] },
    		{ "before": ["enter"], "commands": ["expand_region"] },
    	],
    	"macros": {
    		"openConfigFiles": [
    			"workbench.action.openSettingsJson",
    			"workbench.action.openGlobalKeybindingsFile"
    		]
    	},
    	"debug.console.fontSize": 20,
    	"debug.console.acceptSuggestionOnEnter": "on",
    }
    
    ```

  * 键位配置在`~/.config/User/keybindings.json`:

    ```json
    [
    	{ "key": "ctrl+n", "command": "editor.action.formatDocument" },
    	{ "key": "ctrl+q", "command": "workbench.action.closeActiveEditor" },
    	{ "key": "ctrl+shift+i", "command": "workbench.action.createTerminalEditorSide" },
    	{ "key": "ctrl+i", "command": "workbench.action.createTerminalEditor" },
    	{ "key": "ctrl+[", "command": "workbench.action.previousEditor" },
    	{ "key": "ctrl+]", "command": "workbench.action.nextEditor" },
    	{ "key": "ctrl+h", "command": "workbench.action.focusLeftGroup" },
    	{ "key": "ctrl+j", "command": "workbench.action.focusBelowGroup" },
    	{ "key": "ctrl+k", "command": "workbench.action.focusAboveGroup" },
    	{ "key": "ctrl+l", "command": "workbench.action.focusRightGroup" },
    	{ "key": "ctrl+'", "command": "macros.openConfigFiles" },
    	{ "key": "ctrl+m", "command": "workbench.action.toggleEditorWidths" },
    	{ "key": "ctrl+t", "command": "workbench.action.tasks.runTask" },
    	{ "key": "ctrl+d", "command": "workbench.action.toggleSidebarVisibility" },
    	{ "key": "ctrl+s", "command": "workbench.action.gotoSymbol" },
    	{ "key": "ctrl+x", "command": "workbench.action.debug.selectandstart" },
    	{ "key": "ctrl+g", "command": "workbench.view.scm" },
    	{ "key": "ctrl+a", "command": "search.action.openNewEditorToSide" },
    	{ "key": "ctrl+f", "command": "workbench.action.quickOpen" },
    	{ "key": "ctrl+e", "command": "workbench.view.extensions" },
    	{ "key": "ctrl+z", "command": "workbench.action.toggleZenMode" },
    	{ "key": "ctrl+r", "command": "workbench.action.reloadWindow" },
    	{ "key": "ctrl+w", "command": "workbench.view.explorer" },
    	{ "key": "ctrl+shift+e", "command": "workbench.files.action.collapseExplorerFolders" },
    	{ "key": "ctrl+b", "command": "workbench.action.focusActiveEditorGroup" },
    	{ "key": "ctrl+shift+h", "command": "workbench.action.moveEditorToLeftGroup" },
    	{ "key": "ctrl+shift+j", "command": "workbench.action.moveEditorToBelowGroup" },
    	{ "key": "ctrl+shift+k", "command": "workbench.action.moveEditorToAboveGroup" },
    	{ "key": "ctrl+shift+l", "command": "workbench.action.moveEditorToRightGroup" },
    	{ "key": "tab", "command": "selectNextQuickFix", "when": "editorFocus && quickFixWidgetVisible" },
    	{ "key": "shift+tab", "command": "selectPrevQuickFix", "when": "editorFocus && quickFixWidgetVisible" },
    	{ "key": "tab", "command": "selectNextSuggestion", "when": "editorTextFocus && suggestWidgetMultipleSuggestions && suggestWidgetVisible" },
    	{ "key": "shift+tab", "command": "selectPrevSuggestion", "when": "editorTextFocus && suggestWidgetMultipleSuggestions && suggestWidgetVisible" },
    	{ "key": "tab", "command": "selectNextSuggestion", "when": "inDebugRepl && suggestWidgetVisible" },
        { "key": "shift+tab", "command": "selectPrevSuggestion", "when": "inDebugRepl && suggestWidgetVisible"},
    	{ "key": "ctrl+tab", "command": "toggleSuggestionDetails", "when": "suggestWidgetHasFocusedSuggestion && suggestWidgetVisible && textInputFocus" },
    	{ "key": "ctrl+p", "command": "editor.action.triggerSuggest" }
    ]
    ```

  * vscode task配置模板: `.vscode/tasks.json`

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


### neovim

* 从`apt`直接下载的`neovim`可能不是最新的, 需要添加`ppa`源再下载:

  ```bash
  firefox https://github.com/neovim/neovim/blob/master/INSTALL.md
  ```

* 注意, 如果Ubuntu是Arm64, 那么需要从源码编译neovim, 可以参考:

  ```bash
  firefox https://github.com/neovim/neovim/blob/master/BUILD.md
  ```

* 安装完之后, 下载`LunarVim`作为发行版:

  ```bash
  firefox https://www.lunarvim.org/docs/installation
  ```

  * 下载之后, 自定制的部分在: `~/.config/lvim/config.lua`.
  * 然后可以在`~/zsh_user`中`alias lvim=nvim`.

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

* 配置完之后, 可以借助`mason`下载LSP等插件.



### tmux

* 首先安装`tpm`:

```bash
firefox https://github.com/tmux-plugins/tpm
```

* tmux本地的参考配置如下:

```ini
# Make tmux support 256 color and italic
set -g default-terminal 'xterm-256color'
set -as terminal-overrides ',xterm*:Tc:sitm=\E[3m'

# Window id start with 1
set -g base-index 1
setw -g pane-base-index 1

# Set ctrl+a to be prefix
set -g prefix C-a
bind-key C-a send-prefix

# prefix+r: Source config file
unbind r
bind r source-file ~/.tmux.conf

# prefix+arrow: Resize window using arrow keys
bind -r Down resize-pane -D 5 
bind -r Up resize-pane -U 5 
bind -r Right resize-pane -R 5 
bind -r Left resize-pane -L 5 

# prefix+j/l: Split window down and right
unbind j
unbind l
bind j split-window -v
bind l split-window -h

# prefix+m: Maximize a pane
unbind m
bind m resize-pane -Z

# prefix+n: Create new window
unbind n
bind-key n command-prompt -p "window name:" "new-window; rename-window '%%'"

# prefix+[]: Navigate window
unbind [
unbind ]
bind-key -r [ previous-window
bind-key -r ] next-window

# prefix+I: Install all tmux plugins
set -g @plugin 'tmux-plugins/tpm'

# Plugin1: Enable <C-h/j/k/l> to navigate window
set -g @plugin 'christoomey/vim-tmux-navigator'

# Plugin2: Tmux themes
set -g @plugin 'dracula/tmux'
set -g @dracula-show-powerline true
set -g @dracula-show-left-icon session
set -g @dracula-show-timezone false

# running tmux plugin manager (keep this line at the bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'

```



### Anaconda

* 安装可以采用CLI方式, 参考:

  ```bash
  firefox https://docs.anaconda.com/anaconda/install/
  ```




## Ubuntu远程环境

> 在这个环境下基本没有`sudo`权限.

* 同样先明确一下操作系统和发行版的信息:

  ```bash
  uname -m && cat /etc/*release
  ```

* 首先确保配置了ssh的免密登陆, 并且可以使用vscode远程登录.
* 然后, 安装Anaconda.

### vim

* 服务器一般会有`vim`, `vim`的无插件配置可以如下, 在`~/.vimrc`

  ```ini
  set nu
  set wrap
  set ruler
  set hlsearch
  set autoindent
  set showmatch
  set wildmenu
  set incsearch
  set ignorecase
  set scrolloff=5
  set laststatus=2
  set cursorline
  
  " Cursor
  "" Normal
  let &t_EI.="\e[1 q"
  "" Insert
  let &t_SI.="\e[5 q"
  "" Replace
  let &t_SR.="\e[4 q"
  
  syntax on
  
  nmap W $
  nmap B ^
  vmap W $
  vmap B ^
  
  nmap H 5h
  vmap H 5h
  nmap J 5j
  vmap J 5j
  nmap K 5k
  vmap K 5k
  nmap L 5l
  vmap L 5l
  ```



### 代理

* 如果服务器能够联网, 那么就可以通过端口forwarding使用本地的代理, 一行命令就行:

  ```bash
  ssh -NR 9999:127.0.0.1:15236 myserver
  ```

  * 这个命令会把服务器上的`9999`端口映射到本地的HTTP代理端口`15236`.
  * 在服务器上用`export all_proxy=http://127.0.0.1:9999`即可完成.

### tmux

* 配置文件在`~/.tmux.conf`

  ```ini
  # Set ctrl+a to be prefix
  set -g prefix C-a
  bind-key C-a send-prefix
  
  # prefix+arrow: Resize window using arrow keys
  bind -r Down resize-pane -D 5 
  bind -r Up resize-pane -U 5 
  bind -r Right resize-pane -R 5 
  bind -r Left resize-pane -L 5 
  
  # prefix+n: Create new window
  unbind n
  bind-key n command-prompt -p "window name:" "new-window; rename-window '%%'"
  
  # prefix+[]: Navigate window
  unbind [
  unbind ]
  bind-key -r [ previous-window
  bind-key -r ] next-window
  ```



### vscode远程插件

* 远程安装插件可以采用VSIX的方式.
  * 插件市场: https://marketplace.visualstudio.com/VSCode
* 下载之后, 将`*.vsix`文件上传到服务器, 然后在vscode中用`install from vsix`进行安装.
