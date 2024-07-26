# Chapter 2 : Shell工具与脚本

## 变量
直接使用等号来定义变量，如
```bash
foo=bar
```
在变量前加上`$`来使用变量的值：
```bash
> echo $foo
bar
```
需注意，在shell中，**空格**被固定用作参数的分隔符，故需慎用。如下，shell将会把foo视作一个函数而试图调用它而非变量名进行赋值，然后因实际不存在foo指令而报错。同样的，在处理名字含有空格的文件是也许小心，注意用引号将名字引起来。
```bash
foo = bar
```
#### 字符串
可以使用双引号`"`或单引号`'`定义字符串。对于纯文本字符串而言，二者等价。但对于一些特殊字符串则略有区别，如含有变量的字符串
```bash
> echo "$foo"
bar
> echo '$foo'
$foo
```
## 函数
```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```
这样便定义了一个名为mcd的函数，他会在当前位置新建一个文件夹并进入其中。我们可以将其写入一个文件中，如`mcd.sh`，并在shell中使用`source mcd.sh`来导入它，而后便可以直接调用'mcd'这个函数。
#### 特殊字符
程序中的`$1`是指在调用mcd时输入的第一个参数。在shell中，有一系列以`$`为首的特殊变量作为系统保留字。如`$1`,`$2`...以此类推分别表示输入的第一，二....个参数。又有`$0`表示脚本的名称，`$?`可以获取上一条命令的错误代码，`$_`可以获得上一条命令的最后一个参数。
还有例如`!!`，它可以用来指代上一条命令，在尝试一些缺权限的东西时很有用，比如当直接输入
```bash
> mkdir /mnt/new
mkdir: /mnt/new: Permission denied
```
由于此时不是root权限，在该处创建文件夹会显示如上提示。此时，可以直接使用
```bash
sudo !!
```
便可直接重新以管理员权限执行。

---
### 错误代码
在shell中，除了标准输入流`STDIN`与标准输出流`STDOUT`，还有标准错误流`STDERR`。它用来输出错误信息而不污染输出。在上文中提及，可用`$?`来获取其中上一条命令的错误代码。如
```bash
> echo "Hello"
Hello
> echo $?
0
```
同大多数主流语言一样，shell也使用0来表示正确无错误的指令。类似的，当尝试执行
```bash
> grep foobar mcd.sh
> echo $?
1
```
由于`mcd.sh`中并不存在`foobar`字符串，便不会输出任何东西耳机报出错误代码1
特殊的，`true`命令的错误代码始终为0，而'false'命令的错误代码始终为1

---
### 简单的条件语句
除了if-else之外，我们可以利用一些逻辑运算符完成简单的条件语句。例如逻辑或`||`,只有当前面的命令失败，才会执行后面的命令:
```bash
> false || echo "Oops fail"
Oops fail
> true || echo "Will not be printed "
>
```
逻辑与`&&`，当前一条命令失败时，后面的命令将被跳过。
```bash
> true && echo "Things went well"
Things went well
> false && echo "This will not be printed"
>
```
此外，我们可以通过`;`连接两个命令在一行写出。

---
### 命令替换
使用`$`
```bash
> foo=$(pwd)
```
也可以直接在字符串中展开(注意使用双引号!!)
```bash
> echo "We are in $(pwd)"
We are in /Users/.......
```
---
### 进程替换
命令替换会将命令结果替换为字符串，而进程替换略有不同。它会在内部执行命令，并命令的结果保存为一个临时文件，在将其文件路径替换输入。例如:
```bash
> cat <(ls) <(ls ..)
```
这会将父文件夹与自己执行ls的结果合并再输出。

---
#### ***Example***
```bash
#!/bin/bash
echo "Starting Program at $(date)" # date输出当前日期
echo "Running program $0 with $# arguments with pid $$" # $0为程序名，$#为参数数量，$$为当前程序processID
for file in "$@"; do  # $@ 指所有参数
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is no found, grep has exit status
    # We redirect STDOUT and STDERR to a null register fille about them
    if [[ "$?" -ne 0 ]]:  then
        echo "File $file does not have any foobar adding one"
        echo "# foobar" >> "$file"
    fi
done
```
这段函数将会在给定文件中查找`foobar`，若没有则加上。

`grep`行分别利用`>`与`2>`来重定向STDOUT与STDERR的输出内容，而`/dev/null`是一个特殊位置，导入其中的数据会被直接舍弃（垃圾桶？）
当`grep`正常执行会返回0，否则返回一个非零数，因此可用此判断是否含有特定字符。

`if`中用`$?`获取STDERR，`-ne`表示not equal，视作运算符。
而利用`[[]]`所括则是`test`命令的写法，用来判断表达式是否为真，输出0或1。

---
### `?`与`*`
`?`可指代一个字符替换，例如：
```bash
> ls project?
```
可列举出`project1`,`project2`等文件，而如`project10`则不会被列出

`*`则表示任意替换，例如:
```bash
> ls *.sh
```
可列出所有sh后缀的文件

---
### `{}`缩写
类似`ABC{1,2,3}`的写法将会被展开为`ABC1 ABC2 ABC3`,例如想将一张图片由png转换为jpg，可以:
```bash
> convert image.{png,jpg}
```
就等于
```bash
> convert image.png image.jpg
```
再如在多个类似文件夹中创建多个类似的文件，可以写为
```bash
> touch project{1,3}/dev/disk/python{1,2,3,4,5}.py
```
一次性创建$2 \times 5=10$个文件，岂不美哉?

再进一步可以省略列举如将`{a,b,c,d,e}`写成`{a..e}`进一步简写

---
### 使用其他语言编写脚本
在bash中使用shell编写一些较为复杂的功能不是一个好想法，因此可以使用其他语言编写程序，例如python

```py
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```
这段代码会将输入的多个参数倒序输出。普通的py代码自己没法直接获取参数输入，实现此功能利用了导入的`sys`库，
    而`argv`则类似于shell中`$1` `$2`等等可用来获取参数。

保存这段程序后当然可以通过在终端中输入`python foo.py`来直接调用解释器来执行。
    若是直接输入`foo.py`，终端无法自动确定文件类型并执行，默认为shell。
    这时候文件第一行的Shebang符(`#!`)则发挥作用，其后接上一串绝对路径来指示所用解释器，并在执行时将文件名与其他参数一同以参数的形式提交给解释器执行
    但其使用绝对路径且不会使用环境变量，在不同机器上解释器位置可能不同，此时需若想调用环境变量，可写为:
```py
#!/usr/bin/env python
```
`env`命令用来调用环境变量路径，而他的位置是不变的，因此可以输入参量`python`来让其帮我们找到编译器执行。

---
### shellcheck
shell不像一些现代语言一样容易debug，于是便有shellcheck帮我们自动检查
```bash
> shellcheck mcd.sh
```
---
### tldr
查询详细的命令介绍文档可以用`man`，而`tldr`则会显示更加简短明了的介绍
```bash
> tldr convert
```
---
## 查找
### find
```bash
> find . -name src -type d 
> find . -path '**/test/*.py' -type f
> find . -mtime -1
```
find查找十分有用，可用很多flag，例如:  
`.`:指示查找范围为当前所在位置，find也会向下递归查找  
`-name`:名字  
`-type`:类型，`d`为文件夹，`f`为文件  
`-path`:路径，`**`表示前面套有多层文件夹，`*.py`则查找python文件  
`-mtime`:最后修改时间，-1表示昨天  
etc.  
find还可以在查找后执行一些操作，例如
```bash
> find . -name "*.tmp" -exec rm {} \;
```
他将会找到所有.tmp文件并将其删除  
对于大量查找来说，我们也可以像操作数据库一样维护并利用前缀快捷查找，例如
```bash
> updatedb # update database
> locate missing-semester
```
可以快速找到所有路径含有missing-semester的位置

### grep
```bash
> grep foobar mcd.sh
```
像这样我们可以在单一文件中查找一段内容,但若是想在许多文件中找到含有特定内容的文件呢？  
```bash
> grep -R foobar .
```
`-R`让grep在指定位置向下遍历文件并搜索给定内容  

### rg

可用`rg`命令可以提升搜索美观度
```bash
> rg "import requests" -t py -C --stats 5 ~/scartch
> rg -u --files-without-match "^#\!" -t sh
```
`-t` 指定后缀名，确定文件类型  
`-C` 可显示查找内容周边5行  
`--stats` 显示搜索信息如时间，文件数。
`-u` 显示隐藏文件  
`--files-without-match` 去除满足给定形式的文件，这里`^#\!`表示文件开头为`#!`(Shebang符)

### fzf  
交互式查找
```bash
> cat example.sh | fzf
```
也可与`Ctrl+R`联动来便捷查找.

---
### history
在终端里可以使用上下箭头来翻阅选择历史指令，也可以通过`history`来查看
```bash
> history 1
```
后接数字选择历史指令起始位置，例如1显示所有历史指令  
如果想要查找特定指令，可以结合`grep`，如
```bash
> history 1 | grep convert
```
或者按下`Ctrl+R`来逐个查找便于找到并执行

---
### tree
显示树状文件图
### broot
更美观可交互搜索的文件
### nnn
美观交互文件遍历 
