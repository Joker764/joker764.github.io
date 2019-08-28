---
title: linux 常用命令之文本处理工具
date: 2019-08-27 17:39:46
tags:   
  - linux
  - shell
  - 命令行
categories:
  - 工具
---
linux 中常用的命令行工具，对文本进行处理，有 `grep`、`cut`、`sort`、`uniq`、`tee`、`diff`、`paste`、`tr`，其中最常用的就是 `grep` 和 `cut` 还有更牛逼的 `ack`。

<!-- more -->

## 1. grep

> `grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）`  是一种强大的文本搜索工具，他能使用正则表达式搜索文本，并把匹配的行打印出来。

- 用于过滤/搜索的特定字符
- 可用正则表达式配合使用，使用十分灵活

### 常用选项

```
-i      # 不区分大小写
-v      # 查找不包含指定行的内容，反向选择
-w      # 按单词搜索
-o      # 打印匹配关键字
-n      # 显示行号
-r      # 逐层遍历目录查找
-A      # 显示匹配行以及其后面多少行
-B      # 显示匹配行以及其前面多少行
-C      # 显示匹配行以及其前后多少行
-l      # 只列出匹配的文件名
-L      # 列出不匹配的文件名
-e      # 使用正则表达式
-E      # 使用扩展正则匹配
^key    # 以关键字开头
key$    # 以关键字结尾
^$      # 匹配空行
--color=auto	# 将关键字加上颜色
```

### 正则表达式

```
^           # 锁定行的开始
$           # 锁定行的结尾
.           # 匹配一个非换行符的字符
*           # 0 个或多个
.*          # 任意字符
[]          # 一个范围内的字符，[Gg] 表示 G 或 g
[^]         # 一定范围内以什么开头
\(..\)      # 标记匹配符，\(love\)，此时 love 就被标记为 1
\<          # 锁定单词的开始，匹配包含这个单词的行
\>          # 锁定单词的结尾
x\{m\}      # 重复字符 x，重复了 m 次
x\{m,\}     # 至少重复了 m 次
x\{m,n\}    # 重复了 m～n 次
\w          # 匹配文字和字符串，即 [A-Za-z0-9]
\W          # \w 的取反形式，匹配一个或多个非单词字符，比如点句号等
\b          # 单词锁定符，如 \bjojo\b 表示只匹配 jojo
```

### 常见用法

在文件中搜索一个单词，命令会返回一个包含 "match_pattern" 的文本行：

```shell
$ grep match_pattern file_name
$ grep "match_pattern" file_name
```

在多个文件中查找：

```shell
$ grep "match_pattern" file_1 file_2 file3 ...
```

输出除了目标之外的所有行：

```shell
$ grep -v "match_pattern" file_name
```

使用正则表达式：

```shell
$ grep -E "[1-9]+"
$ egrep "[1-9]+"
```

只输出文件中匹配的部分，主要配合正则使用：

```shell
$ echo this is a test line. | grep -o -E "[a-z]+\."
```

统计文件或文件中包含匹配字符串的行数：

```shell
$ grep -c "text" file_name
```

输出包含匹配字符串的行数：

```shell
$ grep "text" -n file_name
$ cat file_name | grep "text" -n
```

打印样式匹配所位于的字符或字节偏移：

```shell
$ echo gun is not unix | grep -b -o "not"
```

搜索多个文件并查找匹配文本在哪些文件中：

```shell
$ grep -l "text" file1 file2 file3 ...
```

### 递归搜索文件

在多级目录中对文本进行递归搜索：

```shell
$ grep "text" ./ -r -n
```

忽略大小写：

```shell
$ echo "hello world" | grep -i "HELLO"
```

选项 `-e` 可以匹配多个样式：

```shell
$ echo this is a text line | grep -e "is" -e "line" -o
```

也可以使用 `-f` 来匹配多个样式，使用的是样式文件：

```shell
$ cat patfile
aaa
bbb
$ echo aaa bbb ccc ddd eee | grep -f patfile -o
```

在 grep 搜索结果中包括或排除指定文件：

```shell
# 只在目录中所有的 .php 和 .html 文件中递归搜索字符 "main()"
$ grep "main()" . -r --include *.{php,html}

# 在搜索结果中排除所有的 README 文件
$ grep "main()" . -r --exclude "README"

# 在搜索结果中排除 filelist 文件列表中的文件
$ grep "main()" . -r --exclude-from filelsit
```

使用 0 值字节的后缀的 grep 与 xargs：

```shell
# 测试文件
$ echo "aaa" > file1
$ echo "bbb" > file2
$ echo "ccc" > file3

$ grep "aaa" file* -lZ | xargs -0 rm
# 执行后会删除 file1 和 file3，grep 的 -Z 选项是用来指定以 0 值字节为终结的文件名，xargs -0 读取输入并用 0 值字节终结符分割文件名，然后删除匹配文件
```

显示匹配到行之前多少行，之后多少行：

```shell
$ seq 10 | grep 5 -A 3
$ seq 10 | grep 5 -B 3
$ seq 10 | grep 5 -C 3
# 如果匹配结果有多个，中间会以 -- 作为分隔符
```

## 2. cut

- 用于列的截取
- 用于连接文件

### 常用选项

```
-c      # 以字符为单位进行分割截取
-d      # 自定义分隔符，默认为制表符 \t
-f      # 与 -d 一起使用，指定截取哪个区域
```

### 常见用法

```shell
$ cut -d: -f1 test.text      # 以 : 为分隔符的第一列内容
$ cut -d: -f1,6,7 test.text  # 以 : 为分隔符，第 1、6、7 列的内容
$ cut -c4 test.txt           # 截取文件每行中的第 4 个字符
$ cut -c1-4 test.txt         # 截取每行中的 1~4 个字符
$ cut -c5- test.txt          # 截取每行从第 5 个字符开始
$ cut -c-2 test.txt          # 截取每行的前面两个字符
```

### 实例

```shell
$ cat test.txt
No Name Mark Precent
01 tom 69 91
02 jack 71 87
03 alex 68 98

$ cut -f2,3 test.txt
Name Mark
tom 69
jack 71
alex 68
```

