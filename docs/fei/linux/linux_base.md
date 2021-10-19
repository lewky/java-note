# linux基础

## bash shell设置环境变量

通过打开bash shell时执行某个脚本文件来实现

* 所有用户
  * `/etc/profile`(bash login shells)
  * `/etc/bashrc`(bash non-login shells)
* 当前用户
  * `~/.bash_profile`(bash login shells)
  * `~/.bashrc`(bash non-login shells)

>* bash login shells：只在用户登陆时执行
>* bash non-login shells：打开bash shells时执行

## ssh

`ssh-keygen`命令：生成SSH的公钥(`id_rsa.pub`)和私钥(`id_rsa`)

ssh client：`~/.ssh/known_hosts`，存放已信任的ssh server信息

ssh server：`~/.ssh/authorized_keys`，存放已信任的ssh client的公钥

## 进程挂起

`ctrl + z`：挂起当前进程

`jobs`：查看当前会话启动的进程

* `fg %jobNumber`：前台运行进程
* `bg $jobNumber`：后台运行进程

## themes & icons

* 主题`themes`相关文件的存放目录：`/usr/share/themes`或者`~/themes`
* 图标`icons`相关文件的存放目录：`/usr/share/icons`或者`~/.local/share/icons`
