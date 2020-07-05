[toc]

grep 的用法就不多说了，这里只列出常见应用

## 去掉 # 开头的行和 ^$ （即空白行）

```
$ cat /etc/nginx/nginx.conf | grep -vE "#|^$" 
```

其中 grep 命令选项如下：

- -v 是反向选择

- -E 是正则表达式