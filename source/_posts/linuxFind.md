title: linuxFind
date: 2015-09-02 15:03:16
tags: linux命令
---
find是常用的查找命令：
------

find <指定目录> <指定条件> <指定动作>

find  &lt;dir&gt; -name &lt;'fileName Conditiion'&gt; -print：查找文件

find  . -perm -005：根据权限查找，这三个数字分别代表：读、写、执行权限。

find &lt;dir&gt; -path &lt;'dir Conditiion'&gt; -prune -o -print：查找目录，prune指出要忽略的目录；如果同时使用了-depth，会忽略prune参数选项。

find test \\(-path test/test4 -o -path test/test3 \\) -prune -o -name "*.log" -print

>注：\表示引用，即指示 shell 不对后面的字符作特殊解释，留给find命令解释；圆括号表示表达式的结合。

按文件属主查找：
find &lt;-path&gt; -user &lt;userName&gt; -print
find &lt;path&gt; -nouser &lt;userName&gt; -print 可以查找到帐户已删除的文件

find &lt;path&gt; -group &lt;groupName&gt; -print
find &lt;path&gt; -nogroup -print 查找没有有效所属用户组的所有文件

find / -mtime -5 -print 更改时间在5日以内的(+3：距当前三日以前的)  -atime  -ctime

find -newer &lt;fileName&gt; ! -newer &lt;fileName&gt; 比前面文件新，但比后面文件旧的文件。

find &lt;path&gt; -type d  目录查找，l链接文件查找。

find &lt;path&gt; -mount 当前文件系统，不进入其他文件系统查找。

——depath：先匹配所有文件，再在子目录中查找。

其他查找命令：
-----
locate：查找文件。如果有查找不到的新文件，执行updatedb，因为其搜索的是一个/var/lib/locatedb数据库，如果查找不到手动更新数据库。

whereis：只用于程序名的搜索(-b 二进制文件)(-m man说明文件)(-s 源代码文件)，省略参数返回所有信息。

which：在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。

type：用来区分某个命令到底是由shell自带的，还是由shell外部的独立二进制文件提供的；如果是外部提供的，-p参数可以显示命令的路径。

find与xargs：
-------
ind命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部
find . -type f -print | xargs file  查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件

find / -name "core" -print | xargs echo "" >/tmp/core.log  在整个系统中查找内存信息转储文件(core dump) ，把结果保存到/tmp/core.log 文件中

find . -perm -7 -print | xargs chmod o-w  当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限

find . -type f -print | xargs grep "hostname"  grep命令在所有的普通文件中搜索hostname这个词

find . -name "*.log" | xargs -i mv {} test4  xargs执行mv 
>**注：**使用-i参数默认的前面输出用{}代替，-I参数可以指定其他代替字符。

find . -name "*.log" | xargs -p -i mv {}
>**注：**-p参数会提示让你确认是否执行后面的命令,y执行，n不执行。
