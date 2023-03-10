### 重置Linux密码





---

### 修改该权限

chown(change owner):  修改所属用户和组
chmod(change mode):  修改用户的权限

> $ ls -l    显示一个文件的属性以及文件所属的用户和组

```
[centos@192 usr]$ ls -l
总用量 268
dr-xr-xr-x.   2 root root 49152 12月  3 18:39 bin
drwxr-xr-x.   2 root root     6 4月  11 2018 etc
drwxr-xr-x.   2 root root     6 4月  11 2018 games
drwxr-xr-x.  40 root root  8192 12月  3 18:38 include
dr-xr-xr-x.  43 root root  4096 12月  3 18:39 lib
dr-xr-xr-x. 145 root root 81920 12月  3 18:43 lib64
drwxr-xr-x.  49 root root 12288 12月  3 18:39 libexec
drwxr-xr-x.  12 root root   131 12月  3 18:34 local
dr-xr-xr-x.   2 root root 20480 12月  3 18:39 sbin
drwxr-xr-x. 240 root root  8192 12月  3 18:39 share
drwxr-xr-x.   4 root root    34 12月  3 18:34 src
lrwxrwxrwx.   1 root root    10 12月  3 18:34 tmp -> ../var/tmp

```

第一个字符代表文件的属性：
`d`：目录
`-`：文件
`l`：链接文档
`b`：装置文件里的可供存储的借口设备（可随机存取装置）
`c`：装置文件里的串行端口设备，如键盘、鼠标

后面的字符三个位一组，第一组表示属主（该文件所有者）拥有该文件的权限，第二组表示属组（所有者的同组用户）拥有该文件的权限，第三组表示其他用户拥有改文件的权限。
其中`r`表示可读(read)，`w`表示可写(write)，`x`表示可执行(execute)，若没有该权限则用`-`表示。

---

### 更改文件属性

> 更改文件属性
> chgrp [-R] 属组名  文件名
> 参数选项
> -R: 递归更改文件属组，在更改某个目录文件的属组时，如果加上-R参数，那么该目录下所有文件的属组都会更改

> 更改文件属主，也可以同时更改文件属组
> chown [-R] 属主名 文件名
> chown [-R] 属主名： 属组名 文件名

