title: gitBase
date: 2015-09-02 12:33:22
tags: git
---
**git是分布式版本控制系统**，客户端并不只提取最新版本的文件快照，而是把原始的代码仓库完整地镜像下来。如此一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜 像出来的本地仓库恢复。因为每一次的提取操作，实际上都是一次对代码仓库的完整备份。可以指定和若干不同的远端代码仓库进行交互。籍此，你就可以在同一个项目中，分别和不同工作小组的人相互协作。
一、git的工作方式：
------
Git 更像是把变化的文件作快照后，记录在一个微型的文件系统中，保存一个指向这次快照的索引，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一链接。Git近乎所有的操作都是本地化执行，所以操作非常快。

Git使用SHA-1 算法计算数据的校验和，通过对文件的内容或目录的结构计算出一个 SHA-1 哈希值，作为指纹字符串：作索引。该字串由 40 个十六进制字符（0-9 及 a-f）组成。

**git是一套内容寻址文件系统。**
>-1、git init时会创建一个.git目录，几乎git所有存储与操作都位于该目录下，备份或者复制一个库，基本上将这一目录老帖到其他地方就可以了。

>-2、.git目录中，config包含特有的项目配置；.gitignore 文件中管理的忽略模式；

>-3、四个重要的文件或目录：HEAD及index文件，objects(存储所有数据内容)及refs目录(存储指向数据[分支]的提交对象的指针)；HEAD文件指向当前分支，index保存了暂存区的信息。

>-4、git hash-object -w 指示 hash-object 命令存储 (数据) 对象，若不指定这个参数该命令仅仅返回键值。

>-5、git cat-file -p：cat-file 命令可以将数据内容取回，参数p让该命令输出数据内容的类型。

>-6、git cat-file -p 83ba > test.txt：将文件恢复到某一个版本。

>-7、对象存储的并不是文件名而仅仅是文件内容。

二、git区域与搭建git环境：
1、暂存区：已修改，未commit。
2、工作区：已跟踪且有修改，还没add；git checkout将工作区的修改全部撤销，如果暂存区有，则恢复到暂存区的状态；没有到版本库取。
3、工作区未跟踪：未跟踪文件，未纳入git管理；可能通过git clean -f fileName/ -d dirName清除文件或目录
4、原始状态：未跟踪，未修改
5、回退merge：只是merge了，git reset回退；merge以后还有别的操作get revert -m <merge前的版本号>。
6、reset会让commit修改的内容丢失，用git rebase -i HEAD~2，再git commit --amend。

暂存区——>工作区(reset HEAD)
版本库——>原始状态(reset --hard HEAD)
工作区——>原始状态(checkout)
未跟踪——>原始状态(clean -f)


三、git常用命令讲解：
--------------
git init *初始化本地仓库*
git config --global user.name "Some One"
git config --global user.email "someone@gmail.com"
git config --global color.status/ diff / branch等 *配置颜色*
$ git config --global alias.co checkout  *设置命令的别名*
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
>*注：*例如取消暂存，两条命令合并
git unstage fileA
$ git reset HEAD fileA

git clone repo地址 *clone远程仓库*
git status *查看当前版本状态*

git add 'xxx'
git add .

git commit -m 'xxx'   *提交*
git commit --amend -m 'xxx'  *合并上一次提交（用于反复修改）*
git commit -am 'xxx'  *将add和commit合为一步*

git rm xxx    *删除index中的文件**
git rm -r *   *递归删除*

git show dfb02e  *显示某个提交的详细内容*
git show HEAD    *显示HEAD提交日志*
git show HEAD^   *显示HEAD的父（上一个版本）的提交日志 ^^为上两个版本 ^5为上5个版本*

git tag          *显示已存在的tag*
git tag -a v2.0 -m 'xxx'  *增加v2.0的tag*
git show v2.0    *显示v2.0的日志及详细内容*
git log v2.0     *显示v2.0的日志*

git diff         *显示所有未添加至index的变更*
git diff --cached   *显示所有已添加index但还未commit的变更*
git diff HEAD^      *比较与上一个版本的差异*
git diff HEAD -- ./lib   *比较与HEAD版本lib目录的差异*
git diff origin/master..master   *比较远程分支master上有本地分支master上没有的*
git diff origin/master..master --stat  *只显示差异的文件，不显示具体内容*

git remote add origin git+ssh://git@192.168.53.168/VT.git *增加远程定义（用于push/pull/fetch）*
git branch  *显示本地分支*
git branch --contains 50089  *显示包含提交50089的分支*
git branch -a  *显示所有分支*
git branch -r  *显示所有原创分支*
git branch --merged  *显示所有已合并到当前分支的分支*
git branch --no-merged  *显示所有未合并到当前分支的分支*
git branch -m master master_copy  *本地分支改名*
git branch -d hotfixes/BJVEP933  *删除分支hotfixes/BJVEP933（本分支修改已合并到其他分支）*
git branch -D hotfixes/BJVEP933  *强制删除分支hotfixes/BJVEP933*

git checkout -b master_copy          *从当前分支创建新分支master_copy并检出*
git checkout features/performance    *检出已存在的features/performance分支*
git checkout --track hotfixes/BJVEP933  *检出远程分支hotfixes/BJVEP933并创建本地跟踪分支*
git checkout v2.0    *检出版本v2.0*
git checkout -b devel origin/develop  *从远程分支develop创建新本地分支devel并检出*
git checkout -- README   *检出head版本的README文件（可用于修改错误回退）*

git merge origin/master     *合并远程master分支至当前分支*
git cherry-pick ff44785404a8e  *合并提交ff44785404a8e的修改*

git push origin master      *将当前分支push到远程master分支*
git push origin :hotfixes/BJVEP933    *删除远程仓库的hotfixes/BJVEP933分支*
git push --tags     *把所有tag推送到远程仓库*

git fetch              *获取所有远程分支（不更新本地分支，另需merge）*
git fetch --prune      *获取所有原创分支并清除服务器上已删掉的分支*
git pull origin master *获取远程分支master并merge到当前分支*
git mv README README2  *重命名文件README为README2*

git reset --hard HEAD  *将当前版本重置为HEAD（通常用于merge失败回退）*
git rebase
git revert dfb02e6e4f2f7b573337763e5c0013802e392818  *撤销提交dfb02e6e4f2f7b573337763e5c0013802e392818*

git ls-files           *列出git index包含的文件*

git show-branch        *图示当前分支历史*
git show-branch --all  *图示所有分支历史*
git whatchanged        *显示提交历史对应的文件修改*

git ls-tree HEAD    *内部命令：显示某个git对象*
git rev-parse v2.0  *内部命令：显示某个ref对于的SHA1 HASH*

git reflog    *显示所有提交，包括孤立节点*
git log -p -2  *仅显示最近的两次更新提交的内容差异*
git log --pretty=format:'%h %s' --graph  *图示提交日志*
>**--pretty参数值：**
>-oneline 将每个提交放在一行显示
>-short，full 和fuller 展示的信息或多或少有些不同
>-format %H  提交对象（commit）的完整哈希字串
%h  提交对象的简短哈希字串
%T  树对象（tree）的完整哈希字串
%t  树对象的简短哈希字串
%P  父对象（parent）的完整哈希字串
%p  父对象的简短哈希字串
%an 作者（author）的名字
%ae 作者的电子邮件地址
%ad 作者修订日期（可以用 -date= 选项定制格式）
%ar 作者修订日期，按多久以前的方式显示
%cn 提交者(committer)的名字
%ce 提交者的电子邮件地址
%cd 提交日期
%cr 提交日期，按多久以前的方式显示
%s  提交说明

其他参数及说明：
--stat 显示每次更新的文件修改统计信息。
--shortstat 只显示 --stat 中最后的行数修改添加移除统计。
--name-only 仅在提交信息后显示已修改的文件清单。
--name-status 显示新增、修改、删除的文件清单。
--abbrev-commit 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
--relative-date 使用较短的相对时间显示（比如，“2 weeks ago”）。
--graph 显示 ASCII 图形表示的分支合并历史。
-(n)    仅显示最近的 n 条提交
--since, --after 仅显示指定时间之后的提交。
--until, --before 仅显示指定时间之前的提交。
--author 仅显示指定作者相关的提交。
--committer 仅显示指定提交者相关的提交。
如：git log --pretty="%h - %s" --author=gitster --since="2008-10-01" \
   --before="2008-11-01" --no-merges -- t/

git show HEAD~3
git show -s --pretty=raw 2be7fcb476
git show HEAD@{5}
git show master@{yesterday}    *显示master分支昨天的状态                      *

git stash		*暂存当前修改，将所有至为HEAD状态*
git stash list   *查看所有暂存*
git stash show -p stash@{0}  *参考第一次暂存*
git stash apply stash@{0}    *应用第一次暂存*

git grep "delete from"    *文件中搜索文本“delete from”*
git grep -e '#define' --and -e SORT_DIRENT

四、git中的dangling对象：
--------
1、git管理项目时，在提交之前，对文件的每次修改，都进行git add去更新index文件 ，产生很多dangling blob与dangling commit对象。
2、git gc --prune="0 days"或者git prune：在达到一定条件下会清除这些dangling对象，git prune --dry-run是查看。
3、tree对象，记录一个blob对象的SHA-1值，权限，名字等信息；树对象于文件系统中的目录，记录多个文件及目录。
4、git cat-file -t id(前4位)：返回其对象类型tree/blob/commit。
5、检查没有被引用的blob，tree，commit对象
6、git fsck --lost-found  将"悬空"对象写入到.git/lost-found/commit/ 或 .git/lost-found/other/目录中。
:其他参数：
:1、--root   报告当前repo的根节点。
:2、--tags   报告检查过的tags。
:3、--no-reflog   将那些仅被reflog引用的commit视为unreachable。
7、find .git/objects
>如果一个对象已经被pack，那么git prune不会删除。
8、git repack -a -d  会直接删掉pack文件中的dangling对象。
git repack -A -d 并不会立即删除，而是将pack文件中的dangling object重新以松散对象的形式存储在database中。也就是把dangling object从pack文件中"吐出来"。

五：mktree、write-tree、read-tree、commit-tree
tree对象：tree相当于unix的目录，blob大致相当于inodes或文件内容。一个tree对象包含一条或者多条记录，每条记录有一个指向blob或子tree对象的SHA-1指针。
git cat-file -p master^{tree}：分支上最新提交的tree对象。
git cat-file -p：让该命令输出数据内容的类型

自己创建tree：通常根据暂存区或index来创建tree。
git update-index --add --cacheinfo 100644：
创建index，将文件的首个版本添加到一个新的暂存区；
由于原来不在暂存区，故加参数--add；
当前添加的文件 并不在当前目录下，而是数据库中，帮加参数--cacheinfo;
指定文件模式：10044

git write-tree SHA-1 ：命令将暂存区域的内容写到一个 tree 对象，如果tree对象不存在，会自动根据 index 状态创建一个 tree 对象。

read-tree 命令将 tree 对象读到暂存区域中去。在这时，通过传一个 --prefix 参数给 read-tree，将一个已有的 tree 对象作为一个子 tree 读到暂存区域。

使用 commit-tree 命令，指定一个 tree 的 SHA-1，如果有任何前继提交对象，也可以指定。

七：远程服务(git可以有多个远程服务端,可以有多个服务端并能从其中一个（合并工作）读取再写入另一个)
添加：git remote add anotherName Url
本地分支与远程分支比较：git diff master..anotherName/master
查看远程分支HEAD指针的改动：git log remote/branch

八：恢复git rebase
git rebase <commitId>
git reset --hard/--soft HEAD^ 把上面的恢复的提交删除

git cherry-pick

十、git log和git reflog高阶运用：


十一、缓存cached
git reset/rm --cached
git diff --cached