---
title: Vim常用命令
date: 2020-10-19 09:33:40
tags:
- Vim
- Linux
categories:
- Vim
---

## 基本命令

```sh
vimtutor // vim 教程
```

### 移动光标

```vim
# hjkl
# 2w 向前移动两个单词
# 3e 向前移动到第 3 个单词的末尾
# 0 移动到行首
# $ 当前行的末尾
# gg 文件第一行
# G 文件最后一行
# 行号+G 指定行
# <ctrl>+o 跳转回之前的位置
# <ctrl>+i 返回跳转之前的位置
```

### 退出

```vim
# <esc> 进入正常模式
# :q! 不保存退出
# :wq 保存后退出
```

### 删除

```vim
# x 删除当前字符
# dw 删除至当前单词末尾
# de 删除至当前单词末尾，包括当前字符
# d$ 删除至当前行尾
# dd 删除整行
# 2dd 删除两行
```

### 修改

```vim
# i 插入文本
# A 当前行末尾添加
# r 替换当前字符
# o 打开新的一行并进入插入模式
```

### 撤销

```vim
# u 撤销
# <ctrl>+r 取消撤销
复制粘贴剪切
# v 进入可视模式
# y 复制
# p 粘贴
# yy 复制当前行
# dd 剪切当前行
```

### 状态

```vim
#<ctrl>+g 显示当前行以及文件信息
查找
# / 正向查找（n：继续查找，N：相反方向继续查找）
# ？ 逆向查找
# % 查找配对的 {，[，(
# :set ic 忽略大小写
# :set noic 取消忽略大小写
# :set hls 匹配项高亮显示
# :set is 显示部分匹配
```

### 替换

```vim
# :s/old/new 替换该行第一个匹配串
# :s/old/new/g 替换全行的匹配串
# :%s/old/new/g 替换整个文件的匹配串
```

### 折叠

```vim
# zc 折叠
# zC 折叠所有嵌套
# zo 展开折叠
# zO 展开所有折叠嵌套
```

### 执行外部命令

```vim
# :!shell 执行外部命令
```

## .vimrc

.vimrc 是 Vim 的配置文件，需要我们自己创建：

```vim
cd Home               // 进入 Home 目录
touch .vimrc          // 配置文件

# Unix
# vim-plug
# Vim
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
# Neovim
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

[https://github.com/junegunn/vim-plug](https://github.com/junegunn/vim-plug)

[https://github.com/FengShangWuQi/to-vim/blob/master/.vimrc](https://github.com/FengShangWuQi/to-vim/blob/master/.vimrc)

## 基本配置

### 取消备份

```vim
set nobackup
set noswapfile
```

### 文件编码

```vim
setencoding=utf-8
```

### 显示行号

setnumber

### 取消换行

setnowrap

### 显示光标当前位置

setruler

### 设置缩进

```vim
set cindent
set tabstop=2
set shiftwidth=2
```

### 突出显示当前行

setcursorline

### 查找

```vim
set ic
set hls
set is
```

### 左下角显示当前vim模式

setshowmode

### 代码折叠

```vim
#启动 vim 时关闭折叠代码
set nofoldenable
```

### 主题

```vim
syntax enable
set background=dark
colorscheme solarized
```

[https://github.com/altercation/vim-colors-solarized](https://github.com/altercation/vim-colors-solarized)
[https://github.com/Anthony25/gnome-terminal-colors-solarized](https://github.com/Anthony25/gnome-terminal-colors-solarized)

## 插件配置

### 树形目录

```vim
Plug 'scrooloose/nerdtree'
Plug 'jistr/vim-nerdtree-tabs'
Plug 'Xuyuanp/nerdtree-git-plugin'

autocmd vimenter * NERDTree
map <C-n> :NERDTreeToggle<CR>
let NERDTreeShowHidden=1
let g:NERDTreeShowIgnoredStatus = 1
let g:nerdtree_tabs_open_on_console_startup=1
let g:NERDTreeIndicatorMapCustom = {
    \ "Modified"  : "✹",
    \ "Staged"    : "✚",
    \ "Untracked" : "✭",
    \ "Renamed"   : "➜",
    \ "Unmerged"  : "═",
    \ "Deleted"   : "✖",
    \ "Dirty"     : "✗",
    \ "Clean"     : "✔︎",
    \ 'Ignored'   : '☒',
    \ "Unknown"   : "?"
    \ }

# o 打开关闭文件或目录
# e 以文件管理的方式打开选中的目录
# t 在标签页中打开
# T 在标签页中打开，但光标仍然留在 NERDTree
# r 刷新光标所在的目录
# R 刷新当前根路径
# X 收起所有目录
# p 小写，跳转到光标所在的上一级路径
# P 大写，跳转到当前根路径
# J 到第一个节点
# K 到最后一个节点
# I 显示隐藏文件
# m 显示文件操作菜单
# C 将根路径设置为光标所在的目录
# u 设置上级目录为根路径
# ctrl + w + w 光标自动在左右侧窗口切换
# ctrl + w + r 移动当前窗口的布局位置
# :tabc 关闭当前的 tab
# :tabo   关闭所有其他的 tab
# :tabp   前一个 tab
# :tabn   后一个 tab
# gT      前一个 tab
# gt      后一个 tab
```

[https://github.com/scrooloose/nerdtree](https://github.com/scrooloose/nerdtree)
[https://github.com/jistr/vim-nerdtree-tabs](https://github.com/jistr/vim-nerdtree-tabs)
[https://github.com/Xuyuanp/nerdtree-git-plugin](https://github.com/Xuyuanp/nerdtree-git-plugin)

### 代码，引号，路径补全

```vim
Plug 'Valloric/YouCompleteMe'
Plug 'Raimondi/delimitMate'
Plug 'Shougo/deoplete.nvim', { 'do': ':UpdateRemotePlugins' }
```

[https://github.com/Valloric/YouCompleteMe](https://github.com/Valloric/YouCompleteMe)
[https://github.com/Raimondi/delimitMate](https://github.com/Raimondi/delimitMate)
[https://github.com/Shougo/deoplete.nvim](https://github.com/Shougo/deoplete.nvim)

### 语法高亮，检查

```vim
Plug 'sheerun/vim-polyglot'
Plug 'w0rp/ale'

let g:ale_linters = {
\    'javascript': ['eslint'],
\    'css': ['stylelint'],
\}
let g:ale_fixers = {
\    'javascript': ['eslint'],
\    'css': ['stylelint'],
\}
let g:ale_fix_on_save = 1

let g:ale_sign_column_always = 1
let g:ale_sign_error = '●'
let g:ale_sign_warning = '▶'

nmap <silent> <C-k> <Plug>(ale_previous_wrap)
nmap <silent> <C-j> <Plug>(ale_next_wrap)
```

[https://github.com/w0rp/ale](https://github.com/w0rp/ale) 
[https://github.com/sheerun/vim-polyglot](https://github.com/sheerun/vim-polyglot)

### 文件，代码搜索

```vim
Plug 'rking/ag.vim'
Plug 'kien/ctrlp.vim'
```

[https://github.com/kien/ctrlp.vim](https://github.com/kien/ctrlp.vim)
[https://github.com/ggreer/the_silver_searcher](https://github.com/ggreer/the_silver_searcher)
[https://github.com/rking/ag.vim](https://github.com/rking/ag.vim)

### 加强版状态栏

```vim
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'

let g:airline_theme='papercolor'
```

[https://github.com/vim-airline/vim-airline](https://github.com/vim-airline/vim-airline)
[https://github.com/vim-airline/vim-airline-themes](https://github.com/vim-airline/vim-airline-themes)

### 代码注释

```vim
Plug 'scrooloose/nerdcommenter'

# <leader>cc // 注释
# <leader>cm 只用一组符号注释
# <leader>cA 在行尾添加注释
# <leader>c$ /* 注释 */
# <leader>cs /* 块注释 */
# <leader>cy 注释并复制
# <leader>c<space> 注释/取消注释
# <leader>ca 切换　// 和 /* */
# <leader>cu 取消注释

let g:NERDSpaceDelims = 1
let g:NERDDefaultAlign = 'left'
let g:NERDCustomDelimiters = {
            \ 'javascript': { 'left': '//', 'leftAlt': '/**', 'rightAlt': '*/' },
            \ 'less': { 'left': '/**', 'right': '*/' }
        \ }
```


[https://github.com/scrooloose/nerdcommenter](https://github.com/scrooloose/nerdcommenter)

### git

```vim
Plug 'airblade/vim-gitgutter'
Plug 'tpope/vim-fugitive'
```

[https://github.com/airblade/vim-gitgutter](https://github.com/airblade/vim-gitgutter)
[https://github.com/tpope/vim-fugitive](https://github.com/tpope/vim-fugitive)

### Markdown

```vim
Plug 'suan/vim-instant-markdown'

let g:instant_markdown_slow = 1
let g:instant_markdown_autostart = 0
# :InstantMarkdownPreview
```

[https://github.com/suan/vim-instant-markdown](https://github.com/suan/vim-instant-markdown)

### Emmet

```vim
Plug 'mattn/emmet-vim'

let g:user_emmet_leader_key='<Tab>'
let g:user_emmet_settings = {
         \ 'javascript.jsx' : {
            \ 'extends' : 'jsx',
         \ },
      \ }
```

[https://github.com/mattn/emmet-vim](https://github.com/mattn/emmet-vim)

### html 5

```vim
Plug'othree/html5.vim'
```

[https://github.com/othree/html5.vim](https://github.com/othree/html5.vim)

### css 3

```vim
Plug 'hail2u/vim-css3-syntax'
Plug 'ap/vim-css-color'

augroup VimCSS3Syntax
  autocmd!

  autocmd FileType css setlocal iskeyword+=-
augroup END
```

[https://github.com/hail2u/vim-css3-syntax](https://github.com/hail2u/vim-css3-syntax)
[https://github.com/ap/vim-css-color](https://github.com/ap/vim-css-color)

### JavaScipt

```vim
Plug 'pangloss/vim-javascript'
let g:javascript_plugin_jsdoc = 1
let g:javascript_plugin_ngdoc = 1
let g:javascript_plugin_flow = 1
set foldmethod=syntax
let g:javascript_conceal_function             = "ƒ"
let g:javascript_conceal_null                 = "ø"
let g:javascript_conceal_this                 = "@"
let g:javascript_conceal_return               = "⇚"
let g:javascript_conceal_undefined            = "¿"
let g:javascript_conceal_NaN                  = "ℕ"
let g:javascript_conceal_prototype            = "¶"
let g:javascript_conceal_static               = "•"
let g:javascript_conceal_super                = "Ω"
let g:javascript_conceal_arrow_function       = "⇒"
let g:javascript_conceal_noarg_arrow_function = " "
let g:javascript_conceal_underscore_arrow_function = " "
set conceallevel=1
```

[https://github.com/pangloss/vim-javascript](https://github.com/pangloss/vim-javascript)
（注：上述脚本中存在特殊字符，有的情况下显示不正确，请直接用上述链接的内容。）

### React

```vim
Plug 'mxw/vim-jsx'
let g:jsx_ext_required = 0
```

[https://github.com/mxw/vim-jsx](https://github.com/mxw/vim-jsx)

### Prettier

```vim
Plug 'prettier/vim-prettier', {
  \ 'do': 'yarn install',
  \ 'for': ['javascript', 'typescript', 'css', 'less', 'scss', 'json', 'graphql'] }
let g:prettier#config#bracket_spacing = 'true'
let g:prettier#config#jsx_bracket_same_line = 'false'
let g:prettier#autoformat = 0
autocmd BufWritePre *.js,*.jsx,*.mjs,*.ts,*.tsx,*.css,*.less,*.scss,*.json,*.graphql PrettierAsync
# :Prettier
```

[https://github.com/prettier/vim-prettier](https://github.com/prettier/vim-prettier)


转载：

[如何让 vim 成为我们的神器](https://segmentfault.com/a/1190000011466454)
