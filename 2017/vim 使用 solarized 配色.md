## 直接 checkout 代码 ##
```SHELL
git clone git://github.com/altercation/solarized.git
```
这里包括所有的配色方案，如果只想给 vim 使用，也可以使用这个：Vim Repository

## 复制 `solarized.vim` 到 `.vim/colors` 目录 ##
```SHELL
cd vim-colors-solarized/colors
cp solarized.vim ~/.vim/colors/
```
## 更改 ~/.vimrc ##
```SHELL
syntax enable

if has('gui_running')
    set background=light
else
    set background=dark
endif

colorscheme solarized
```
### 有两个主题，也可以只用一种 ###
**light**
```SHELL
syntax enable
set background=light
colorscheme solarized
```
**dark**
```SHELL
syntax enable
set background=dark
colorscheme solarized
```
