# 基础操作

## 打开和编辑文件

**edit and read**

如果打开的文件不存在，则会新建。

`vim 文件1 文件2 文件n -O` 打开多个文件，以并排方式显示
`vim 文件1 文件2 文件n -o` 打开多个文件，以堆叠方式显示
`vim -x 文件名` 打开加密文件（需要输入密码）

`:browse oldfiles` 显示最近打开文件
`:ls` 查看缓冲区打开的文件
`:buffer N` 打开缓冲区中序号为N的文件

`:e 文件名` 打开文件
`:e!` 撤销编辑当前文件或指定文件

`:r 文件名` 读取指定文件内容，并将其插入到光标后面。
`:r! 外部命令` 读取外部命令标准输出，并将其插入到光标后面。
`:r! echo %` 插入当前文件名
`:r! echo %:p` 插入当前全路径名

`<c-g>` 或 `:f` 显示文件名、文件状态、光标位置、总行数、进度
`:f 文件名` 更改当前文件的文件名。最后需要 `:w` 保存为新文件，否则不生效

## 保存和退出

**write and quit**

`:q` 退出
`:q!` 强制退出，`!` 表示强制
`:qa` 退出所有

`:w` 保存
`:w 文件名` 另存为
`:wq` 保存并退出

**推荐使用以下命令替代上面的 `:wq` 和 `:q!`**

`:x` 保存并退出（同 `:wq`，但是只有做出修改才会保存）

`ZZ` 保存并退出，和 `:x` 相同
`ZQ` 作废退出，和 `:q!` 相同

## 移动、滚动、跳转

**motion, scroll , jump**

`h` 左移 `j` 下移 `k` 上移 `l` 右移
`(` 和 `)` 句子间移动
`{` 和 `}` 段落间移动

`w` 右移到单词开头（word）
`b` 左移单词开头（back）
`e` 右移单词结尾（end）
`ge` 左移到单词结尾（go end）



`0` 或 `Home` 行首
`$` 或 `End` 行末
`^` 移动到行首非空字符
`gg` 或 `<c-Home>` 文档首行 
`G` - `<c-End>` 文档结尾
`{count}G` 跳到第n行 
`{count}gg` 跳到第n行



`H` - High（当前屏幕最顶行）
`M` - Medium（当前屏幕中间行）
`L` - Low（当前屏幕最底行）
`zz` 将当前行置为屏幕中间



滚动时光标是不会移动的：
`<c-e>` 向下滚动一行
`<c-y>` 向上滚动一行
`<c-u>` 向下滚动小半屏
`<c-d>` 向上滚动小半屏
`<c-f>` 或 `PgDn` 向下翻页
`<c-b>` 或 `PgUp` 向上翻页



`gi` 回到上一次编辑的地方并插入
`<c-t>` 或 `<c-o>` 回到上一次光标位置
`<c-i>` 回到前一次光标位置

`<c-]>` 跳转到一个对象

### 行内搜索移动

`f{char}` 右移到 char 字符上
`F{char}` 左移到 char 字符上
`t{char}` 
`T{char}` 

`;` 和 `,` 搜索该行上面操作的下一个和上一个

### 标记跳转

**mark**

`m[a-zA-Z]` 标记当前位置，然后任意位置按
``[a-zA-Z]` 跳转到标记位置
`'[a-zA-z]` 跳转到标记位置的行首


### vim-easymotion 瞬移大法

1. 安装 [vim-easymotion](https://github.com/easymotion/vim-easymotion) 插件：`Plug 'easymotion/vim-easymotion'`

2. 添加映射启用插件

   ```vimscript
   " 添加 ss 映射启用
   nmap ss <Plug>(easymotion-s2)
   ```

3. Normal 模式下按下 ss，然后输入两个目标字符，这时屏幕会高亮显示目标字符，最后按下对应的字母就能瞬间跳过去

4. `:help easymotion` 查看帮助文档



easymotion 插件如下：

- easymotion-prefix：easymotion 前缀按键插件
- easymotion-bd-f 或 easymotion-s： bidirectional find|search，双向查找并移动插件
- easymotion-bd-f2 或 easymotion-s2： bidirectional find|search 2-character，双向查找两个字符并移动插件
- easymotion-bd-wl 或 easymotion-sl：bidirectional within line，双向行移动插件
- easymotion-bd-jk： bidirectional jk，双向上下行移动插件
- easymotion-bd-w：bidirectional word，双向字移动插件
- easymotion-bd-tl：
- easymotion-t2：
- easymotion-overwin-f：over window find，多窗口查找移动插件
- easymotion-overwin-f2：over window find 2-character，多窗口查找两个字符移动插件



参考：

https://zhuanlan.zhihu.com/p/89844330

https://www.imooc.com/video/19466

## 插入

**insert and append**

`i` insert 插入到光标前
`I` 插入到行首
`a` append 插入到光标后
`A` 插入到行尾
`o` open 在光标下面插入新行
`O` 在光标上面插入新行
`gi` 回到上一次编辑的地方进行插入

## 删除更改

**substitute, delete, change, replace**

注意：vim 会将删除的内容保存到寄存器，以便进行粘贴

`x` 或 `dl` 或 `<Del>` 删除光标后的一个字符
`X` 或 `dh` 删除光标前的一个字符

`s` 删除光标后的一个字符，并进入插入模式，相当于 `x` 的增强版
`S` 删除整行并进入插入模式，相当于 `cc`



`d{motion}` 删除
`D` 删除至行尾，相当于 `d$` 
`dd` 删除整行

`c{motion}` 删除并进入插入模式，相当于 `d` 的增强版
`C` 删除至行尾并进入插入模式，相当于 `c$` 
`cc` 删除整行并进入插入模式，相当于 `S`



`r` 替换光标后的一个字符
`R` 进入替换模式，连续替换后面的字符



插入模式下：

`<c-w>` 删除左边的单词

`<c-u>` 删除整行

### vim-surround 成对编辑

https://github.com/tpope/vim-surround

有以下操作：

- cs (change a surrounding)。比如 `cs"'` 将双引号改为单引号；`cs'<q>` 将单引号改为 `<q>` ；`cs]{` 将方括号改为花括号，其中 `{` 有空格，`}` 没空格；
  - cst (change a surrounding tag to)。比如 `cst"` 将当前文本对象的标记比如 `<q>` 改为双引号。
- ds (delete a surrounding)。比如 `ds"` 删除双引号。
- ys (you add a surrounding)。比如 `ysiw]` (`iw` 是文本对象) 添加方括号；`yssb` 或 `yss)` 给整个句子添加圆括号。

需要先将光标移到成对的**文本对象**中，然后进行操作。

## 复制粘贴

**copy and paste**

注意：vim 会将复制的内容保存到寄存器，以便进行粘贴

`y` 复制
`yy` 复制整行
`yiw` 复制光标所在单词

`p` 粘贴至光标后
`P` 粘贴至光标前

小技巧：通过 `xp` 能快速将两个字符位置互换

### *解决粘贴时缩进混乱

在开启了 `:set autoindent` 后，粘贴代码时可能会缩进混乱，这时候使用 `:set paste` 和 `:set nopaste` 来解决

## 撤销重做

**undeo and redo**

`u` 撤销
`<c-r>` 重做

`U` 撤销整行


## 可视化选择模式

**visual**

`v` 可视化选择模式

`V` 可视化行选择模式
`<c-v>` 或 `<c-q>` 可视化列选择模式



另外还可以这样

`v{motion} :command` ：执行命令行。比如 `v{motion} :w filename` 选择文本并另存为 filename 文件

`v{motion} normal operator` ：执行操作。比如  `v{motion} normal @a` 选择文本并回放寄存器a的宏操作


## 查找

`/` 向下查找 
`?` 向上查找

`n` 下一个匹配项（查找模式下）
`N` 上一个匹配项（查找模式下）

`:set hlsearch` 可以设置查找项高亮显示



`%` 匹配括号并移动到括号

`*` 或 `#` 当前单词的前向或后向匹配并跳转
`g* ` 或 `g#` 同上，但却是当前单词的部分匹配并跳转

### *fzf.vim 模糊搜索

https://github.com/junegunn/fzf.vim

首先安装 fzf

```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

然后安装 fzf.vim 插件



`:Files` 命令可以替代 Ctrl-P 插件



**fzf.vim 的依赖**

- [fzf](https://github.com/junegunn/fzf) 0.23.0 or above
- For syntax-highlighted preview, install [bat](https://github.com/sharkdp/bat)
- `Ag` requires [The Silver Searcher (ag)](https://github.com/ggreer/the_silver_searcher)
- `Rg` requires [ripgrep (rg)](https://github.com/BurntSushi/ripgrep)
- `Tags` and `Helptags` require Perl



## 替换

`:s/old/new/`（将行中首个old替换成new）
`:s/old/new/gc`（将行中所有old替换成new，替换时提示确认）
`:#,#s/old/new/g`（将两行内所有old替换成new）

上面命令中最后的斜杠后面是标志符：

- `g` global，表示全局范围执行
- `c` confirm，表示提示确认
- `n` number，表示报告匹配的次数而不替换，适合用来查询匹配的次数



`:%s/old/new/g`（将文中所有old替换成new）

`:n,ms/old/new/`（替换第 n 行开始到 m 行中每一行的第一个 old 为 new）
`:n,ms/old/new/g`（替换第 n 行开始到 m 行中每一行所有 old 为 new）

`:n,$s/old/new/`（替换第 n 行开始到最后一行中每一行的第一个 old 为 new）
`:n,$s/old/new/g`（替换第 n 行开始到最后一行中每一行所有 old 为 new）

### *far.vim 批量替换

https://github.com/brooth/far.vim



## 组合操作

- count motion：即 次数+方向
- operator motion：即 操作命令+方向
- operator count motion：即 操作命令+次数+方向
- count operator count motion：即 次数+操作命令+次数+方向

**含义解释**

- count：表示操作次数
- motion：表示光标移动方向，比如 `h` 左移 `j` 下移 `k` 上移 `l` 右移
- operator：表示操作命令，比如 `c` 更改 `d` 删除 `y` 复制

## 设置选项

**option**

`:set` 显示所有更改过的设定值
`:set {option}?` 显示指定的设定值
`:set no{option}` 取消指定的设定值
`:help option-list` 显示所有设定值简短说明

`:set autowrite` 设置自动存档，默认关闭
`:set backup` 设置自动备份，默认关闭

`:set mouse=a` 开启鼠标模式
`:set mouse=` 关闭鼠标模式

`:set background=dark` 或 `light`，设置背景风格

`:set cindent` 设置C语言风格缩进

## 重复执行上一次命令

`.` 重复上一次 Normal 模式下的操作（不重复 Command-line 模式下的操作）

### *repeat.vim 插件

地址：https://github.com/tpope/vim-repeat

## *执行外部命令

## 其它

`~` 光标后的一个字符大小写互转
`gU` uppercase 大写
`gu` lowercase 小写

`:ver` 显示vim版本和参数

## 没啥用的命令

`<c-e>` 
`<c-y>` 
`<c-f>` 
`<c-b>` 
`<c-a>` 
`<c-c>` 
`<c-m>` 
`<c-n>` 
`<c-h>` `<c-j>` `<c-k>` `<c-l>` 映射为窗口跳转
`<c-p>` CtrlP 插件在用
`<c-c>` 
`<c-s>` 
`<c-t>` 

`t` 跳转

## 帮助

帮助文档官网：https://vimhelp.org/

`:help` 打开帮助手册
`:help change.txt` 打开参考手册
`:help quickref.txt` 打开快速参考指南

| WHAT                 | 前缀 | 示例                | 备注                                        |
| -------------------- | ---- | ------------------- | ------------------------------------------- |
| Normal mode command  |      | `:help x`           | 查看 Normal 模式下按下 `x` 的作用           |
| Visual mode command  | v_   | `:help v_u`         | 查看 Visual 模式下按下 `u` 的作用           |
| Insert mode command  | i_   | `:help i_<Esc>`     | 查看插入模式下按下 `<Esc>` 的作用           |
| Command-line command | :    | `:help :quit`       | 查看 Command-line 模式下 `:q` 的作用        |
| Command-line editing | c_   | `:help c_CTRL-D`    | 查看 Command-line 模式下按下 `<c-d>` 的作用 |
| Vim 命令参数         | -    | `:help -r`          | 查看 `vim -r` 的作用                        |
| Option 设置          | '    | `:help 'textwidth'` |                                             |
| 正则表达式（新增）   | /    | `:help /[`          |                                             |

# 高级操作

## 插件管理

这里使用 vim-plug 作为插件管理器。项目地址为：https://github.com/junegunn/vim-plug

使用教程

1. 首先安装 vim-plug
2. 编辑 ~/.vimrc 文件
3. 示例文件：
```vimscipt
" 这只是个实例文件
" Specify a directory for plugins
" - For Neovim: stdpath('data') . '/plugged'
" - Avoid using standard Vim directory names like 'plugin'
call plug#begin('~/.vim/plugged')

" Make sure you use single quotes

" Shorthand notation; fetches https://github.com/junegunn/vim-easy-align
" Plug 'junegunn/vim-easy-align'

" Any valid git URL is allowed
" Plug 'https://github.com/junegunn/vim-github-dashboard.git'

" Multiple Plug commands can be written in a single line using | separators
" Plug 'SirVer/ultisnips' | Plug 'honza/vim-snippets'

" On-demand loading
" Plug 'scrooloose/nerdtree', { 'on':  'NERDTreeToggle' }
" Plug 'tpope/vim-fireplace', { 'for': 'clojure' }

" Using a non-default branch
" Plug 'rdnetto/YCM-Generator', { 'branch': 'stable' }

" Using a tagged release; wildcard allowed (requires git 1.9.2 or above)
" Plug 'fatih/vim-go', { 'tag': '*' }

" Plugin options
" Plug 'nsf/gocode', { 'tag': 'v.20150303', 'rtp': 'vim' }

" Plugin outside ~/.vim/plugged with post-update hook
" Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }

" Unmanaged plugin (manually installed and updated)
" Plug '~/my-prototype-plugin'

" Initialize plugin system
call plug#end()
```

3. `:source ~/.vimrc` 让系统重新加载 .vimrc 配置文件

4. `:PlugInstall` 安装插件

## 代码补全

**Completion**

在插入模式下：

`<c-n>` 或 `c-p` 关键字补全。会根据上下文有哪些单词进行补全。n是next，p是previous的意思。

`<c-x><c-f>` 补全文件名

`<c-x><c-o>` 补全代码，需要开启文件类型检查，安装插件

`:help ins-completion` 查看插入模式下的代码补全帮助文档



在 Command-line 模式下：

`Tab` 补全命令
`<c-d>` 显示补全列表

### 代码补全插件

https://github.com/ycm-core/YouCompleteMe

https://github.com/maralla/completor.vim

## 代码注释

添加注释

1. `<c-v>` 进入可视化列选择模式
2. `I` 插入到行首，输入注释符 `//` 或 `#`
3. 按下 `ESC`（两下）

取消注释

1. `<c-v>` 进入可视化列选择模式
2. 选择行首注释符 `//` 或 `#`
3. `d` 删除

### vim-commentary 代码注释

https://github.com/tpope/vim-commentary

`gcc` 注释当前行
`gc{motion}` 注释，比如 `gcag` 注释一个自然段
Visual 模式也可以使用 `gc` 注释高亮文本

### nerdcommenter 代码注释

https://github.com/preservim/nerdcommenter

## 代码格式化与缩进

`:help formatting` 查看格式化与缩进的帮助文档

`>>` 整行向右缩进
`<<` 整行向左缩进

通过 `:set shiftwidth=4` 可以设置缩进的字符数，默认是8



`:ce` - center 使本行内容居中
`:ri` - right 使本行内容靠右
`:le` - left 使本行内容靠左



`={motion}` 自动缩进（并不是真正的格式化）
`==` 自动缩进当前行
`={` 格式化代码块
`gg=G` 全文格式化
`mG=nG` 自动缩进第m到n行



有时候自带的缩进功能并不好用，这时候可借助

`:!cmd` 执行外部命令进行格式化

比如调用外部命令格式化JSON文件：

```shell
:%!python -m json.tool #调用外部python命令，执行json.tool模块的功能进行格式化
```

### 格式化插件

https://github.com/Chiel92/vim-autoformat

## 文件目录树

### NerdTree

https://github.com/preservim/nerdtree

`:NERDTreeToggle` 打开文件目录树
`:NERDTreeFind` 显示当前文件在目录的位置

### CtrlP 模糊搜索

https://github.com/ctrlpvim/ctrlp.vim

## 文本对象

`:help text-objects` 查看文本对象的文档

`iw` inner word
`is` inner sentence
`ip` inner paragraph
`it` inner tag（tag就是一对标签，比如 `<html></html>` ）
`i(` 或 `i{` 或 `i[` 或 `i<` inner block
`i`` ` 或 `i'` 或 `i"` 引号括起来的字符串

## 寄存器

`"{register}` 引用相应的寄存器。比如 `"a` 是引用寄存器a，然后 `yy` 是将整行保存到寄存器a中，最后 `"ap` 是将寄存器a中的内容粘贴出来。

`:registers` 显示寄存器列表。

`:help registers` 查看寄存器相关的帮助文档。

有以下这些寄存器：

- `""` 未命名寄存器，默认使用的寄存器
- `"0` - `"9` 数字命名寄存器
- `"a` - `"z` ， `"A` - `"Z` 字母命名寄存器
- `"+` 系统剪切板寄存器
- `"%` 当前文件名
- `".` 上次插入的文本

## 宏

**recording**

`q` 录制宏，结束录制宏
`q{register}` 录制宏并保存到指定的寄存器中
`@{register}` 回放宏
`:help q` 查看宏相关的帮助文档

## 缓冲区

`:ls` 查看缓冲区打开的文件
`:b n` 或 `:buffer N` 打开缓冲区中序号为N的文件
`:help :ls` 查看缓冲区相关的帮助文档

## 多窗口

**window**

`<c-w,c-n>` 或 `:new` 打开一个新的 vim 视窗（同上，但是容易和系统的快捷键冲突）

`:sp 文件名` 水平分屏来编辑文件
`:vs 文件名` 垂直分屏来编辑文件

`<c-w> s` 或 `:sp` 水平分屏（split）
`<c-w> v` 或 `:vs` 垂直分屏（vertical split）
`<c-w> q` 即 :q 结束分割出来的视窗。如果在新视窗中有输入需要使用强制符！即:q!
`<c-w> o` 打开一个视窗并且隐藏之前的所有视窗
`<c-w> j` 移至下面视窗
`<c-w> k` 移至上面视窗
`<c-w> h` 移至左边视窗
`<c-w> l` 移至右边视窗
`<c-w> J` 将当前视窗移至下面
`<c-w> K` 将当前视窗移至上面
`<c-w> H` 将当前视窗移至左边
`<c-w> L` 将当前视窗移至右边
`<c-w> -` 减小视窗的高度
`<c-w> +` 增加视窗的高度

## 映射

**mapping**

`:help map-modes` 查看映射模式

- `:map` 、`:noremap` 、`unmap` 定义 Normal、Insert 模式下的递归映射、非递归映射、取消映射
- `:nmap` 、`:nnoremap` 、`nunmap` 定义 Normal 模式递归映射、非递归映射、取消映射
- `:vmap` 、`:vnoremap` 、`vunmap` 定义 Visual 模式递归映射、非递归映射、取消映射
- `:imap` 、`:inoremap` 、`iunmap` 定义 Insert 模式递归映射、非递归映射、取消映射

任何时候都应该使用**非递归映射**！

## Git插件

https://github.com/tpope/vim-fugitive

## *Vimscript

参考：https://learnvimscriptthehardway.stevelosh.com/（中文名：笨方法学Vimscript）

# 美化

## 更换配色方案

1. 显示当前配色方案 `:colorscheme`， 默认是 default
2. 列出所有配色方案 `:colorscheme <c-d>`
3. 选择配色方案 `:colorscheme 配色方案名`

网上也有很多配色方案插件。

## vim-startify 启动页

https://github.com/mhinz/vim-startify

`:Startify` 进入启动页

通过以下命令可以查看相关文档

```
:h startify
:h startify-faq
```

## 状态栏和缩进线

**statusline/tabline**

https://github.com/vim-airline/vim-airline

https://github.com/powerline/powerline

**indentline**

https://github.com/Yggdroot/indentLine

# vim vs neovim

| vim                      | neovim             | 备注 |
| ------------------------ | ------------------ | ---- |
| 历史悠久，代码老旧       | 代码新             |      |
| 更新慢                   | 更新快，开发更活跃 |      |
| 不支持异步（Vim8后支持） | 支持异步           |      |


# vimrc 配置文件

1. 打开 vim

2. 编辑 vimrc 文件

   - Linux：`vim ~/.vimrc` 或 `:e ~/.vimrc`  

   - Windows：`:e $VIM/_vimrc` （`$VIM` 也可以替换为 `$HOME`）

3. 读取 vimrc 示例文件的内容，将其插入到光标后

   `:r $VIMRUNTIME/vimrc_example.vim`

4. 自定义您的 vimrc 文件并保存

   - Linux：`:w ~/.vimrc`  
   - Windows：`:w $VIM/_vimrc`

5. 使 vimrc 立即生效

   - Linux：`:source ~/.vimrc`  
   - Windows：`:source $VIM/_vimrc`

我的配置文件（在 $VIMRUNTIME/vimrc_example.vim 的基础上增加配置）：

```
source $VIMRUNTIME/vimrc_example.vim
"""""""""""""""""""""
"   PluginInstall   "
"""""""""""""""""""""
call plug#begin('~/.vim/plugged')
Plug 'mhinz/vim-startify'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'Yggdroot/indentLine'
Plug 'scrooloose/nerdtree'
Plug 'ctrlpvim/ctrlp.vim'
Plug 'easymotion/vim-easymotion'
Plug 'tpope/vim-repeat'
Plug 'tpope/vim-surround'
Plug 'tpope/vim-commentary'
Plug 'tpope/vim-fugitive'
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }
Plug 'junegunn/fzf.vim'
Plug 'brooth/far.vim'
call plug#end()

""""""""""""""""""""""
"      Settings      "
""""""""""""""""""""""
colorscheme elflord
set encoding=gbk	"防止中文乱码
set number			"显示行号
set shiftwidth=4	"默认自动缩进是8，这里改为4
set ignorecase		"查找时忽略大小写
set autoindent		"自动缩进
set pastetoggle=<F2>	"按F2进入粘贴模式
"set foldmethod=indent	"设置折叠方式
set lines=40		"设置GVim窗口高度
set columns=120		"设置GVim窗口宽度

""""""""""""""""""""""
"      Mappings      "
""""""""""""""""""""""
" 配置leader键映射
let mapleader=','
let g:mapleader=','

" 让虚拟行上下移动更快点——Visual linewise up and down by default (and use gj gk to go quicker)
noremap <Up> gk
noremap <Down> gj
noremap j gj
noremap k gk

" 居中查找的下一行或上一行
nnoremap n nzzzv
nnoremap N Nzzzv

" 复制到行尾。使其和 D 和 C 保持一致
nnoremap Y y$

" 使用 leader+w 直接保存
inoremap <leader>w <Esc>:w<CR>
noremap <leader>w :w<CR>

" 使用 jj 进入 Normal 模式并保持光标位置不变
inoremap jj <Esc>`^

" 使用 ctrl+h/j/k/l 切换窗口
noremap <C-h> <C-w>h
noremap <C-j> <C-w>j
noremap <C-k> <C-w>k
noremap <C-l> <C-w>l

" 切换 [b 和 [n 切换 buffer
nnoremap <silent> [b :bprevious<CR>
nnoremap <silent> [n :bnext<CR>

" 格式化JSON
com! FormatJSON %!python3 -m json.tool

"""""""""""""""""""""
"      Plugins      "
"""""""""""""""""""""
nnoremap <Leader>v :NERDTreeFind<CR>
nnoremap <Leader>g :NERDTreeToggle<CR>
let NERDTreeShowHidden=1

" 添加 Ctrl-P 映射开启 CtrlP
let g:ctrlp_map = '<c-p>'
let g:ctrlp_cmd = 'CtrlP'

" Disable default mappings
let g:EasyMotion_do_mapping = 0 
" Turn on case-insensitive feature
let g:EasyMotion_smartcase = 1

" s{char}{char} to move to {char}{char}
nmap <Leader>s <Plug>(easymotion-overwin-f2)

" Move to line
map <Leader>f <Plug>(easymotion-bd-jk)
nmap <Leader>f <Plug>(easymotion-overwin-line)

" JK motions: Line motions
map <Leader>j <Plug>(easymotion-j)
map <Leader>k <Plug>(easymotion-k)

" n-character search motion
map  / <Plug>(easymotion-sn)
omap / <Plug>(easymotion-tn)
map  n <Plug>(easymotion-next)
map  N <Plug>(easymotion-prev)

```

参考配置：https://github.com/fatih/vim-go-tutorial/blob/master/vimrc