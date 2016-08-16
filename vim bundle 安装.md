## 关于 Vundle ##
Vundle 是 Vim 的插件管理器。
Vundle 可以做的事情……
- 可在 .vimrc 文件中跟踪和配置插件
- 安装配置插件
- 更新已安装插件
- 按名字搜索所有可用 Vim 脚本
- 清理不使用的插件
- 再单独的交互界面完成上述操作

Vundle 自动做……
- 管理你安装脚本的路径
- 安装与更新之后重新生成帮助标签
Vundle正在经历一个接口的改变，请保持最新状态，以获得最新的变化。

![Vundle 界面](https://camo.githubusercontent.com/bc559468e6623d18947ced1ef353f68f6116e45a/687474703a2f2f692e696d6775722e636f6d2f527565683743632e706e67)

## 快速开始 ##
- 安装git，因为需要使用git 下载 Vundle。当然也可以直接上 github 下载压缩包，手工部署位置。
- 下载 Vundle
        $ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
- 配置插件：
        将下面的配置信息放置在 .vimrc 文件的最开始位置。移除你不需要的插件，它们只是为了达到说明的目的。
```XML
set nocompatible              " be iMproved, required
filetype off                  " required

" 设置 Vundle 路径，用于 Vundle 运行和初始化
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" 可选，Vundle 安装插件的路径
"call vundle#begin('~/some/path/here')

" 使用 Vundle 管理产检，必须
Plugin 'VundleVim/Vundle.vim'

" 以下是 Vundle 支持管理不同配置格式的例子
" 插件配置必须在 vundle#begin/end 之间
" GitHub 仓库中的插件
Plugin 'tpope/vim-fugitive'
" 来自 http://vim-scripts.org/vim/scripts.html 的插件
Plugin 'L9'
" 没有发布到 github 的git 插件
Plugin 'git://git.wincent.com/command-t.git'
" 本机git 仓库的插件(i.e. 当你开发你自己的插件时使用到)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" 安装 L9 避免命名冲突，如果你已经安装了不同版本的软件
Plugin 'ascenator/L9', {'name': 'newL9'}

" 你的所有插件配置必须在下面这一行之前
call vundle#end()            " required
filetype plugin indent on    " required
" 忽略插件缩进改变，使用如下命令：
"filetype plugin on
"
" 简要帮助
" :PluginList       - 罗列出所有已安装的插件
" :PluginInstall    - 安装插件，加上前缀 '!' 或者 :PluginUpdate 命令，用于更新插件
" :PluginSearch foo - 搜索 foo 插件，加上前缀 '!'，用于清空本地缓存
" :PluginClean      - 确认移除未使用的插件；加上前缀 '!'，用于自动移除未使用的插件
"
" 使用 :h 命令查看 vundle 更多的细节或者 FAQ 的wiki
" 与插件无法的信息可以放置在这行之下
```
## 安装插件 ##
启动 vim 并使用命令 :PluginInstall
通过命令行安装：vim +PluginInstall +qall

## 文档 ##
输入命令 :h vundle 可以看到更多细节

