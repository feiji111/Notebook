# Version Control



# 1. Version Control System

有三种类型的Version Control System(VCS)

## 1.1 Local Version Control Systems

GNU RCS属于这种类型的VCS。

<img src="assets/image-20240813134837295.png" alt="image-20240813134837295" style="zoom:50%;" />



## 1.2 Centralized Version Control Systems

CVS，Subversion(Apache Subversion)和Perforce都属于这种类型。

<img src="assets/image-20240813134902419.png" alt="image-20240813134902419" style="zoom:50%;" />

## 1.3 Distributed Version Control Systems

Git，Mercurial和Darcs属于这种类型。

<img src="assets/image-20240813135033538.png" alt="image-20240813135033538" style="zoom:50%;" />



# 2. Git



## 2.1 A Short History of Git

参考：

- [git的前世，和BitKeeper](https://juejin.cn/post/7130545347767566367)

Git的出现与Linux内核是分不开的。

最早Linux采用的其实是和unix一样的邮件互发patches与archived files的方式来共享更改(1991–2002)。

而从2002开始，Linux kernel开始采用一种DVCS，叫做BitKeeper。但是与Linux的开源相反，BitKeeper是一个商业软件，但是BitKeeper允许Linux社区可以免费使用。

但是在2005年，因为种种原因，BitKeeper收回了对开源社区的免费授权，促使Linux社区开发自己的VCS，git就由此诞生。至于更多详细而细节的内容，可以参考上面的文章。



## 2.2 Basic Knowledge

<img src="assets/v2-3bc9d5f2c49a713c776e69676d7d56c5_r.jpg" alt="img" style="zoom: 50%;" />



相比于其它VCS比如CVS，Subversion以及Perforce，git以一种完全不同的方式存储历史数据。

### 2.2.1 Snapshots, Not Differences

CVS，Subversion以及Perforce采用的是delta-based version control，即记录的是文件的改动。

<img src="assets/image-20240814140129155.png" alt="image-20240814140129155" style="zoom:80%;" />

而git记录的，是文件系统的一个快照(snapshot)。

```
With Git, every time you commit, or save the state of your project, Git basically takes a picture of what all your files look like at that moment and stores a reference to that snapshot. To be efficient, if files have not changed, Git doesn’t store the file again,
just a link to the previous identical file it has already stored.
This makes Git more like a mini filesystem with some incredibly powerful tools built on
top of it, rather than simply a VCS.
```

在git的眼中，这些历史数据就被串成了stream of snapshots。

因此从某种程度上来说，Git就像是一个小型的文件系统。

<img src="assets/image-20240814140447809.png" alt="image-20240814140447809" style="zoom:80%;" />

### 2.2.2 Nearly Every Operation Is Local



### 2.2.3 Git Has Integrity

在Git中，几乎



git采用SHA-1



### 2.2.4 Git Generally Only Adds Data

git中的几乎所有操作都只会向Git的database中添加数据。这就意味着git中的很多操作都是可以撤销的。



### 2.2.5 The Three States

git中，文件有三种状态：modified，staged，committed

这三种状态就分别对应着一个git项目中的三个部分：the working tree，the staging area，the
Git directory

<img src="assets/image-20240814204600565.png" alt="image-20240814204600565" style="zoom:50%;" />



### 2.2.6 Git config

git有许多环境变量可以配置，这些配置信息会被存储在三个地方：

1. `[path]/etc/gitconfig `系统级别的配置文件，这是`--system`选项作用的位置
2. `~/.gitconfig`或者`~/.config/git/config` 用户级别的配置文件，这是`--global`选项作用的位置
3. 项目文件夹下`.git`目录下的`config` 这是`--local`选项作用的位置，也是默认的选项

按照优先级[3] > [2] > [1]，优先级高的选项会覆盖优先级低的选项。

```
git config --list --show-origin
```

用以显示上述三个层级的git配置选项。

```
git config --list
```

也是一样的作用。

但是也可以通过

```
git config <key>
git config --show-origin <key>
```

来显示指定key的配置。

### 2.2.7 

git中，文件有几种状态：

- tracked
  - unmodified
  - modified
  - staged
- untracked



<img src="assets/image-20240815220059127.png" alt="image-20240815220059127" style="zoom: 50%;" />





## 1.2 git workflow





# 3. Github

本文写于2024-8-14，下午1点50分。

之所以写这一章节，是因为在这之前，对于github的认知程度非常浅，但随着不断学习Git以及接触FOSS，逐渐认识到Github有许多更加强大以及方便的功能。

## 3.1 







# 2. SVN



# 3. CVS



# 4.