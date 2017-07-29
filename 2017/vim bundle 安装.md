## 安装 vundle ##
`vundle` 依赖于 `Git`，所以首先得要将 `Git` 安装完成后并配置环境变量，从官网 `Clone Vundle` 到路径 `~/.vim/bundle/Vundle.vim` 下，`~` 是指 `home` 目录，我的 `home` 路径为 `C:\Users\xiaoheike\`，建议不用修改，修改了反而麻烦。
```json
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

## 配置vundle ##
在 `_vimrc` 文件中加入下面的配置

"=====Vundle 插件管理工具=====
set nocompatible                                "禁用 Vi 兼容模式  
filetype off                                    "启用文件类型侦测  

" 如果安装的路径改变了，则这里的路径也要改变
set rtp+=set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" 建议不要配置这个路径，如果匹配，路径也不要设置在 vim 的 plugins 目录（bundle 会无法发现匹配的插件）
" call vundle#begin('~/some/path/here')
   
" 使用Vundle来管理插件，这个必须要有。  
Plugin 'VundleVim/Vundle.vim' 
   
" 以下为要安装或更新的插件

" 所有的插件都在这个之前匹配
call vundle#end()
filetype plugin indent on


基本命令
Vundle常用指令
:PluginList 列出已经安装的插件
:PluginInstall 安装所有配置文件中的插件
:PluginInstall 安装所有配置文件中的插件
:PluginInstall! 更新所有插件
:PluginSearch pluginName 搜索插件
:PluginClean! 根据配置文件删除插件

 
