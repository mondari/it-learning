[TOC]

## vim 命令

`vim 文件1 文件2 文件n -O` 打开多个文件，以并排方式显示
`vim 文件1 文件2 文件n -o` 打开多个文件，以堆叠方式显示

`:ls` 查看缓冲区打开的文件



`vim -x 文件名` 创建加密文档（需要输入密码）

## 四种操作组合（核心）

count motion

operator motion

operator count motion

count operator count motion

**含义解释**

count：代表操作次数

motion：代表光标移动方向，比如 h左移/j下移/k上移/l右移

operator：代表操作命令，比如 c（change）、d（delete）、y（copy）

## 移动 motion

`h` 左移 `j` 下移 `k` 上移 `l` 右移



`w` - word 移动到下一个单词的开头

`ge` - go end 移动到上一个单词的末尾

`b` - back 移动到当前或上一个单词开头

`e` - end 移动到当前或下一个单词结尾



`0` - `Home` 行首

`$` - `End` 行末

`gg` - `CTRL-Home` 文档首行 

`G` - `CTRL-End` 文档结尾

`Ctrl-g` 显示文件名、总行数、光标位置

`{count}G` 跳到第n行 

`{count}gg` 跳到第n行



`H` - High（当前屏幕最顶行）

`M` - Medium（当前屏幕中间行）

`L` - Low（当前屏幕最底行）

## 滚动 定位

`CTRL-u` 向下滚动小半屏

`CTRL-d` 向上滚动小半屏

`Ctrl-f` 向下翻页

`Ctrl-b` 向上翻页



`CTRL-o` 回到上一次光标位置

`CTRL-i` 回到前一次光标位置

`CTRL-t` 回到上一次光标位置

## 插入

i - insert 插入光标前

I - 插入到行首

a - append 插入到光标后

A - 插入到行尾

o - open 在光标下面插入新行

O - 在光标上面插入新行

gi - 回到上一次编辑的地方进行插入

`C-w` 删除上一个单词（插入模式下）

`C-u` 删除整行（插入模式下）



## 删除修改

x - 删除光标后的一个字符

X - 删除光标前的一个字符

s - substitute 删除光标后的一个字符，并进入插入模式，相当于 `x` 的增强版
S - 删除整行并进入插入模式，相当于 `cc`



d - delete 删除

D - 删除至行尾，相当于 `d$` 

dd - 删除整行

c - change 删除并进入插入模式，相当于 `d` 的增强版

C - 删除至行尾并进入插入模式，相当于 `c$` 

cc - 删除整行并进入插入模式，相当于 `S`



r - replace 替换光标后的一个字符
R - 进入替换模式，连续替换后面的字符

## 复制剪切粘贴

y - yank 复制

p - paste 粘贴至光标后

P - 粘贴至光标前

yy - 复制整行





 

## 选择

`v` visual 可视化选择模式

`V` 可视化行选择模式

`C-v` 可视化列选择模式



v {motion} :command

如v {motion} :w filename 选择并另存为



 

## 撤销重做

u - undo 撤销

U 修复整行

`CTRL-r` - redo 重做，注意这个很容易跟 `r` - replace 替换 混淆，



## 查找

`/` 向下查找 

`?` 向上查找 （和 `/` 同一个键位）

`:set hlsearch` 可以设置查找项高亮显示

`n` - next 下一个匹配项（查找模式下）

`N` 上一个匹配项（查找模式下）



% 括号配对



`\*` 寻找游标所在处的单词
`\# ` 同上，但 `\#` 是向前（上）找，`\*` 则是向后（下）找
`g\* ` 同 `\*` ，但部分符合该单词即可
`g\#` 同 `\#` ，但部分符合该单词即可

## 替换

`:s/old/new/`（将行中首个old替换成new）

`:s/old/new/g`（将行中所有old替换成new）

`:s/old/new/gc`（将行中所有old替换成new，交互式替换）

`:#,#s/old/new/g`（将两行内所有old替换成new）



`:%s/old/new/g`（将文中所有old替换成new）

`:%s/old/new/gc`（将文中所有old替换成new，交互式替换）



`:n,ms/old/new/`（替换第 n 行开始到 m 行中每一行的第一个 old 为 new）

`:n,ms/old/new/g`（替换第 n 行开始到 m 行中每一行所有 old 为 new）



`:n,$s/old/new/`（替换第 n 行开始到最后一行中每一行的第一个 old 为 new）

`:n,$s/old/new/g`（替换第 n 行开始到最后一行中每一行所有 old 为 new）

## 缩进 格式化

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


## 其它

`C-w` 切换窗口

`.` 重复上一次普通模式下的操作（不重复命令模式下的操作）

`~` 光标后的一个字符大小写互转

`:ver` 显示vim版本和参数



## 保存 另存为 退出 打开

`:q` - quit 退出

`:q!` - 作废退出，`!` 表示强制

`:qa` - 退出所有

`:w` - write 保存

`:w 文件名` - 另存为

`:wq` - 保存并退出

**推荐使用以下命令替代上面的 `:wq` 和 `:q!`**

`:x` - 保存并退出（同 `:wq`，但是只有做出修改才会保存）

`ZZ` - 保存并退出，和 `:x` 相同

`ZQ` - 作废退出，和 `:q!` 相同



`:e 文件名` - edit 打开新文件

`:e!` - 撤销编辑当前文件或指定文件



`:f` 显示当前编辑文件的文件名

`:f 文件名` 修改当前编辑文件的文件名，但是实际上并没有修改



`:r 文件名` 读取指定文件内容，并将其插入到光标后面。

`:r! 外部命令` 读取外部命令标准输出，并将其插入到光标后面。

`:r! echo %` 插入当前文件名
`:r! echo %:p` 插入当前全路径名

## 帮助

`:help` 先通过此命令查看如何获取具体的帮助！



## 命令补全

`C-n` next 插入时补全

`C-p` previous 插入时补全

`CTRL-D` 显示补全列表（命令模式下）

`Tab` 补全命令（命令模式下）

## 多窗口

`:new` 打开一个新的 vim 视窗
`CTRL-w` 打开一个新的 vim 视窗（同上，但是容易和系统的快捷键冲突）

`:sp` split 水平分屏
`:vs` vertical split 垂直分屏
`:sp 文件名` 打开新的水平分屏视窗来编辑文件
`:vs 文件名` 打开新的垂直分屏视窗来编辑文件

`CTRL-w s` 将当前窗口分割成两个水平的窗口

`CTRL-w v` 将当前窗口分割成两个垂直的窗口

`CTRL-w q` 即 :q 结束分割出来的视窗。如果在新视窗中有输入需要使用强制符！即:q!

`CTRL-w o` 打开一个视窗并且隐藏之前的所有视窗

`CTRL-w j` 移至下面视窗

`CTRL-w k` 移至上面视窗

`CTRL-w h` 移至左边视窗

`CTRL-w l` 移至右边视窗

`CTRL-w J` 将当前视窗移至下面

`CTRL-w K` 将当前视窗移至上面

`CTRL-w H` 将当前视窗移至左边

`CTRL-w L` 将当前视窗移至右边

`CTRL-w -` 减小视窗的高度

`CTRL-w +` 增加视窗的高度

## 设置 option

`:set` 显示所有修改过的设定值

`:set all` 显示所有的设定值

`:set option?` 显示指定的设定值

`:set nooption` 取消指定的设定值

`:set autowrite` 设置自动存档，默认未打开

`:set background=dark` 或 `light`，设置背景风格

`:set backup` 设置自动备份，默认未打开

`: set cindent` 设置C语言风格缩进

# vimrc 配置文件

1. 打开 vim

2. 编辑 vimrc 文件

   - 如果你用的是Linux系统：`:e ~/.vimrc`  

   - 如果你用的是Windows系统：`:e $VIM/_vimrc`

3. 读取 vimrc 示例文件的内容

   `:r $VIMRUNTIME/vimrc_example.vim`

4. 自定义您的 vimrc 文件并保存

5. 使 vimrc 立即生效

   - 如果你用的是Linux系统：`:source ~/.vimrc`  
   - 如果你用的是Windows系统：`:source $VIM/_vimrc`



我的配置文件：

```
"该 vimrc 是在 $VIMRUNTIME/vimrc_example.vim 的基础上添加option
set number    	"显示行号
set shiftwidth=4	"默认自动缩进是8，这里改为4
set ignorecase		"查找时忽略大小写
```