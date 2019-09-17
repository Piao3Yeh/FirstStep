# hashcat的例子
## 使用hash-indentifier确认hash类型

hashcat内置很很多种加密破解算法, 根据需要破解的hash类型选择破解模式.

此次要破解的内容为MySQL5的密码hash, 为了保险起见可以使用hash-indentifier来尝试确认要破解的hash到底是哪一种类型


这次我们把从MySQL数据库的user表的password字段的hash值直接放到hash-indentifier中, 
结果显示为
- SHA-1
- MySQL-SHA-1

可以看到SHA-1 和MySQL-SHA-1虽然都是SHA-1但是有区别, 所以我们这里确认是MySQL-SHA-1

## 使用hashcat 破解
使用`hashcat --help`查看使用方式, 其信息中模式类别中显示 MySQL-SHA算法对应的模式数值为300 .破解方式中字典破解对应的数值为0

我使用`-m 300`指定hash对应的算法, 使用`-a 0` 设为字典爆破模式, 例如
`hashcat -a 0 -m 300 需要被破解的hash值  字典的路径`

其中字典可以自己生成也可以去网上找弱密码字典, 字典文件里面的格式为一行一个密码

## 可能遇到的问题
hashcat 和 hash-indentifier都已经内置到名kali的Linux系统中, 可以直接用, 或者自行安装.

如过kali以虚拟机方式安装, 由于hashcat需要用GPU来提升速度, 所以在虚拟机会报错`Device #1: Not a native Intel OpenCL runtime`

可以在 命令中添加`--force`来忽略此部分, 从而可以正常运行hashcat.
