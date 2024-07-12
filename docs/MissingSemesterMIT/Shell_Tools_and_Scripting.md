# Chapter 2 : Shell Tools and Scripting
## 变量
可以直接使用等号定义变量
```bash
foo=bar
```
**注意**: 在shell中**空格**被用作分隔符(如分隔命令与参数)，应当慎用，如下是反例
```bash
foo = bar
```
shell会将foo认作命令名并试图执行因而报错。
