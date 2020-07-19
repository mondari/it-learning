[toc]



## Shell 脚本入门

### 简单尝试

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

### 添加变量

hello.sh

```shell
#!/bin/bash
hello="hello, world"
echo $hello
```

执行：

```bash
$ bash hello.sh
"hello, world"
```


### 传入参数

hello.sh

```shell
#!/bin/bash
echo $1
```

`$1` 为我们传入的第一个参数

执行：

```bash
$ bash hello.sh "hello, world"
"hello, world"
```

### 使用函数

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


