[toc]

grep 的用法就不多说了，这里只列出常见应用

## 去掉 # 开头的行和空白行

```
cat /etc/nginx/nginx.conf | grep -vE "#|^$" 
```

其中 grep 命令选项如下：

- -v 是反向选择
- -E 是正则表达式

正则表达式中的 `^$` 表示空白行。

## 匹配多个关键字

```bash
grep -E "word1|word2|word3" file.txt
```

参考：https://blog.csdn.net/lijing742180/article/details/84959963

## 搜索指定目录下的文件

```bash
grep -rn 'keyword' /path/to/search
```

其中：

- -r 表示递归查找文件夹下的所有文件
- -n 表示显示行号

参考：https://blog.csdn.net/tsxw24/article/details/7828357

