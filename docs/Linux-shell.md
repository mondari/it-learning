# Shell 脚本入门

建议使用 IDEA 来编辑 Shell 文件。

## 简单尝试

hello.sh

```bash
#!/bin/bash
echo "hello, world"
```

执行：

```bash
$ bash hello.sh
"hello, world"
```

## 添加变量

hello.sh

```shell
#!/bin/bash
hello="hello, world"
echo "$hello"
```

执行：

```bash
$ bash hello.sh
"hello, world"
```


## 传入参数

hello.sh

```shell
#!/bin/bash
echo "$1"
```

`$1` 为我们传入的第一个参数

执行：

```bash
$ bash hello.sh "hello, world"
"hello, world"
```

## 使用函数

hello.sh

```shell
#!/bin/bash
function hello {
  echo "hello, world"
}

hello
```

执行：

```bash
$ bash hello.sh
"hello, world"
```



## 条件语句

### 对文件进行判断

```shell
#!/usr/bin/env bash

if [ -f "$1" ]; then
  echo "file $1 exists"
fi

if [ -x "$1" ]; then
  echo "file $1 is executable"
fi

if [ -r "$1" ]; then
  echo "file $1 is readable"
fi

if [ -w "$1" ]; then
  echo "file $1 is writable"
fi

if [ -s "$1" ]; then
  echo "file $1 is not empty"
fi

#注意多条件表达式使用两个中括号
if [[ -f "$1" && -f "$2" ]]; then
  if [ "$1" -ef "$2" ]; then
    echo "file $1 equals $2"
  elif [ "$1" -nt "$2" ]; then
    echo "file $1 is newer than $2"
  elif [ "$1" -ot "$2" ]; then
    echo "file $1 is older than $2"
  else
    echo "this line will never be executed!"
  fi
fi
```

### 对数字进行条件判断

```shell
#!/usr/bin/env bash

a=0
b=5

if [ $a -eq $b ]; then
   echo "$a equals $b" 
fi

if [ $a -ne $b ]; then
   echo "$a not equals $b" 
fi

if [ $b -gt $a ]; then
    echo "$b is greater than $a"
fi

if [ $a -lt $b ]; then
    echo "$a is less than $b"
fi

if [ $b -ge $a ]; then
    echo "$b is greater than or equal $a"
fi

if [ $a -le $b ]; then
    echo "$a is less than or equal $b"
fi
```

打印结果如下：

```cmd
0 not equals 5
5 is greater than 0
0 is less than 5
5 is greater than or equal 0
0 is less than or equal 5
```

### 对字符串进行判断

```shell
#!/usr/bin/env bash

a=""
b="b"
c="cd"
d="cd"

if [ -z "$a" ]; then
  echo "string a is empty"
fi

if [ -n "$b" ]; then
  echo "string $b is not empty"
fi

if [ "$c" == "$d" ]; then
  echo "string $c equals $d"
fi

if [ "$b" != $c ]; then
  echo "string $b not equals $c"
fi

```

执行结果：

```cmd
string a is empty
string b is not empty
string cd equals cd
string b not equals cd
```



### 其它判断

```shell
#!/usr/bin/env bash

if [ $(command -v ls) ]; then
  echo "exists command ls"
fi

if [ -d "/tmp" ]; then
  echo "exists directory /tmp"
fi

if [ -e "/tmp" ]; then
    echo "exists path /tmp"
fi
```



## 循环语句

```shell
#!/usr/bin/env bash

for (( i = 0; i < 5; i++ )); do
    echo "$i"
done
```

或

```shell
#!/usr/bin/env bash

for i in {1..5} ; do
    echo "$i"
done
```



```shell
#!/usr/bin/env bash

a=5
b=0
# 循环直到 a>b 条件不再满足
while [ $a -gt $b ]; do
  b=$((b + 1))
  echo "$b"
done
```

或

```shell
#!/usr/bin/env bash

a=5
b=0
until [ $a -le $b ]; do
  b=$((b + 1))
  echo "$b"
done
```



## 选择语句

```shell
#!/usr/bin/env bash

echo 'Input a number between 1 to 3: '
read -r aNum
case $aNum in
1)
  echo 'You select 1'
  ;;
2)
  echo 'You select 2'
  ;;
3)
  echo 'You select 3'
  ;;
*)
  echo 'You do not select a number between 1 to 3'
  ;;
esac

```

参考：http://c.biancheng.net/cpp/view/7006.html

## Shell 快捷键

`Ctrl+w`：删除光标前面到空格的内容，并将其保存至 **kill-ring**（剪切环，用来保存删除的内容）

`Ctrl+h`：删除光标前面字符，相当于 `Backspace`

`Ctrl+d`：删除光标后面字符，相当于 `Delete`

**`Alt+d`**：删除光标至单词末尾的内容

**`Ctrl+u`**：清除光标至行首的所有内容

**`Ctrl+k`**：清除光标至行尾的所有内容



**`Alt+f`**：将光标向右移动一个单词

**`Alt+b`**：将光标向左移动一个单词

`Ctrl+f`：将光标右移一个字符，相当于 `->`，这样可以不用将手挪到键盘方向键区

`Ctrl+b`：将光标左移一个字符，相当于 `<-`，这样可以不用将手挪到键盘方向键区

`Ctrl+a`：将光标移动到行首，相当于 `Home`，这样可以不用将手挪到键盘右边功能区

`Ctrl+e`：将光标移动到行尾，相当于 `End`，这样可以不用将手挪到键盘右边功能区



`Ctrl+t`：交换(**transpose**)光标前的两个字符

`Alt+t` 或 `Esc+t`： 交换(**transpose**)光标位置前的两个单词

`Alt+u`：uppercase（转大写）光标至单词末尾的内容

`Alt+l`：lowercase（转小写）光标至单词末尾的内容

`Alt+c`：capitalize（首字母大写）光标至单词末尾的内容



**`Ctrl+-`**：undo，取消刚才的操作

`Crtl+l` ：清屏，相当于执行 `clear` 命令

`Shift+Insert`：将剪贴板内容复制到光标处



`Crtl+y` ：粘贴 **kill-ring**（剪切环，用来保存删除的内容）第一项内容。通常在使用 `Ctrl+w` 误删命令行中的内容，通过 `Crtl+y` 来恢复，或通过 `Ctrl+-` 取消操作。

`Alt+y` ：将 **kill-ring**（剪切环，用来保存删除的内容）中第一项内容移到最后，再粘贴新的第一项内容。通常是在使用 `Crtl+y` 粘贴后，才能使用 `Alt+y` 切换第二项内容并粘贴。

`Alt+.` 或 `Alt+_`：粘贴上一个命令参数

`Ctrl+p`：命令历史中的上一个命令

`Ctrl+n`：命令历史中的下一个命令

`Ctrl+r`：反向搜索命令历史。这样可以快速找到最近执行的命令，搜索过程中按回车可快速执行命令，按 `Esc` 则将命令粘贴到命令行中



参考：

https://www.gnu.org/software/bash/manual/html_node/Commands-For-Moving.html

https://www.gnu.org/software/bash/manual/html_node/Commands-For-History.html

https://www.gnu.org/software/bash/manual/html_node/Commands-For-Text.html

https://www.gnu.org/software/bash/manual/html_node/Commands-For-Killing.html

https://www.gnu.org/software/bash/manual/html_node/Miscellaneous-Commands.html