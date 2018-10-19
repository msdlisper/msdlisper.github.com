---
layout: post
title:  "linux的权限系统"
date:   2018-10-10 09:19:00
categories: learn 
tags:  linux auth
---

* content
{:toc}

linux的权限管理, 群组概念



## 背景
作为前端开发, 相比运维来说, linux可能没有那么重要, 但作为一个有技术追求的开发者来说, 技多不压身, 学习linux是一个拓宽技术的好方向. 总结这篇文章主要原因是在部署测试环境时, 有时候发现nginx不能正常访问静态资源, 当然是权限的问题, 但chmod, gpasswd, usermod这些命令, 真的了解多少? 如何在测试环境正确的模拟线上的用户权限? 他们都是为linux的权限系统服务的, 于是我学习了[鸟哥的Linux私房菜](http://cn.linux.vbird.org/linux_basic/0410accountmanager.php#account) 文件权限和群主概念的相关内容, 然后总结下, 写出blog.

### 场景模拟
>新建的nginx服务, 没法访问报failed (13: Permission denied), 如何排查?

- 查看nginx部署静态文件的路径和文件权限, 使用`ls -al`, 确认文件的'其他用户'的权限是否可读, 以及目录是否可写
    + `chmod -R o=rx ./static` 最简单的办法就是将'其他用户'的权限加上可读,可执行

>使用者 pro1, pro2, pro3 是同一个项目计划的开发人员，我想要让这三个用户在同一个目录底下工作， 但这三个用户还是拥有自己的家目录与基本的私有群组

- 创建用户组
    + `groupadd quickPass`
- 创建用户, 给出初始密码和指定有效用户组
    + `useradd -G quickPass pro1`
    + `useradd -G quickPass pro2```
    + `useradd -G quickPass pro3`
    + `echo "0000" | passwd --stdin pro1`
    + `echo "0000" | passwd --stdin pro2`
    + `echo "0000" | passwd --stdin pro3`
- 创建目录, 只允许quickPass的用户可编辑, 其他可读
    + `mkdir /home/work`
    + `chgrp quickPass /home/work`
    + `chmod 2770 /home/work`

## 概念

### 文件权限

命令 `ls -al`

得到:

`drwx------(文件权限)   3(链接数)       root(owner)       root(所属群组)     4096(大小,字节)   Sep  5 10:37(最后修改时间) .gconf `

上面.gconf目录的权限是: 只有拥有者能读,写,执行. 权值600

其中, '文件权限' : '- --- --- ---' 对应 '类型 拥有者 所属群主 其他人'

其中, 类型: d代表目录, -代表文件, l代表link

权限一般是3种: rwx; r可读4, w可写2, x可执行1

还有三种特殊权限: 

>set Uid: SUID

- 参考`/usr/bin/passwd`, 如果要使用chmod来赋予t权限, 可以使用类似:`chmod 4770 filename`
- 符合的条件:
    + `/usr/bin/passwd` 权限: `-rwsr-xr-x 1 root root`, 被修改的文件只有root才可以操作
    + `/usr/bin/passwd`是二进制
- 能达到的目的:
    + 赋予普通用户拥有这个程序拥有者的权限, 修改类似`/etc/shadow`文件  
- 类比 `/bin/cat`
    + 如果不改变成t权限, 普通用户是不能cat /etc/shadow 文件的
    + 可也改下试试: `chmod 4755 /bin/cat`

>set Gid: SGID

- 参考`/usr/bin/locate`, sgid可以针对目录
- 符合条件:
    + `/usr/bin/locate` 权限: `-rwx--s--x 1 root slocate`, 被修改的文件slocate能使用, 比如`/var/lib/mlocate/mlocate.db`
    + `/usr/bin/locate` 是二进制
- 能达到的目的:
    + 赋予普通用户拥有这个程序拥有组的权限, 比如查看`/var/lib/mlocate/mlocate.db`
- 如果一个目录有SGID权限, 为了是进入该目录的用户

>Sticky Bit: SBIT

- 用户组或其他用户如果有这个目录的wx权限, 并拥有SBIT权限, 那他只能删除自己创建的文件

### 初始群组/有效群组

只要用户加入了某个组, 就会拥有这些群组的权限, 不过, 次数这个组叫次要群组, 还不算有效群组

**初始群组**: passwd后面的gid, 一登录就会取得这个群组的权限. 



**有效群组**: `groups`命令可以列出来所有支持的群组, 通过`newgrp` 来切换, 让用户拥有这个群组的权限, 这些群组是保存在`/etc/gshadow`.


比如`touch abc`, 这个时候abc文件的所属群组就是有效群组. 如果没用`newgrp`切换, 这个有效群组==初始群组



## 涉及到的文件

### /etc/passwd
存放用户信息的文件

`root:x:0:0:root:/root:/bin/bash `

依次解释:

     账号名称, 口令, uid,gid, 用户信息说明栏, 家目录, shell
     口令: 后来到 /etc/shadow了, 没有用, 口令就是登录密码
     uid
     gid, 与文件/etc/group有关, 代表初始群组
     说明
     家目录
     shell

### /etc/shadow

里面有加密后的口令, 如果口令忘了可以用 `passwd`修改


### /etc/group
存放分组信息,`daemon:x:2:root,bin,daemon` 依次: `组名, 口令, gid, 支持的账号`

使用useradd, 会同时增加一个同名的group

### /etc/gshadow
群组密码, 这个文件和`/etc/group` 比较像, 特别是支持的组的部分
`daemon:::root,bin,daemon`, 依次: `组名, 口令, 群组管理员账号, 支持的组`

口令如果是!, 表示没有群组管理员

## 常用的命令

### chmod
改变文件的权限

- `chmod +x filename`
    + 表示给run.sh加可写权限, 效果等于 chmod a+x
    + 同理可以 chmod -x
- `chmod 777 filename`
    + 得到权限`-rwxrwxrwx` 
- `chmod u=rwx,go=rw filename`
    + 得到权限`-rwxrw-rw-`

### useradd
增加用户, 会修改 `/etc/passwd, /etc/shadow, /etc/group, /etc/gshadow`, 并在/home加一个目录

- `useradd xx`
    + 新增用户xx
- `useradd -G usergroup username`
    + 新增用户username, 并加入usergroup
-  类比 `userdel xx` 删除某个用户


### usermod
修改用户权限, 

- `usermod -G user4[,othergorup] user5`
    + 让user5 在 user4组里
    + 修改 `/etc/group`
        * `user4:x:503:user5`
    + 修改`/etc/gshadow`
        * `user4:!::user5`
    + 类比: `gpasswd -a user5 user4`
- `usermod -s /bin/sh user4`
    + 指定登录的shell
    
### passwd
- `passwd`  
    + 用户可以改变自己的密码
- `passwd oneuser`
    + root 改变某个用户的权限
    
### gpasswd
- `gpasswd groupname`
    + 给群组groupname赋予密码
- `gpasswd -A user1,user2 groupname`
    + 给群组groupname赋予管理员

下面是群组管理员的操作:

- `gpasswd -a user5 user4`
    + 让user5 在 user4组里
- `gpasswd -d user5 user4`
    + 让user5 从 user4移出

### newgrp
- `newgrp user4`
    + 将有效组切换到user4
    + 修改? /etc/group, /etc/passwd都没改变
    + 怎样做到的: 另外以一个 shell 来提供这个功能,新的 shell 给user5的有效GID是user4, 可以exit退出此功能

### finger
- 查看有哪些用户登录了, 以及查看具体的用户信息

### id
- 看uid, gid, 有效组的id

