---
date: 2015-08-16T23:04:24+08:00
description: ""
tags:
- golang
- development
- environment
- engineering
title: 利用 git hook 搭建自动编译部署测试环境
topics:
- Development
- golang
---
# 利用 git hook 搭建自动编译部署测试环境
@(Inbox)[golang|开发环境|工程]	


### 说明：
利用 docker，git 搭建自动编译部署测试环境
**步骤：**
1. 在 `dev_server`上 搭建测试环境
2. 创建在 `ci_server`代码远端仓库 `test_repo`
3. 在远端仓库中添加 post-recieve hook
3. 在开发机上本地仓库上新添加一个远程仓库指向的远端仓库

**说明：**
`dev_server`和 `ci_server`都是docker container

**使用说明：**
每次 commit 后
1. 在测试服务器上自动编译
2. 在测试服务器上自动部署

## 步骤说明
1. 创建`dev_server`搭建测试环境：
- 新建一个 container将程序能正常运行 的环境全部部署好，然后将该 container 保存为image，
- 创建相关的 Dockfile
创建 Dockfile 并不是一个必须的步骤，但是创建测试环境的 Dockfile，并保存，能让开发人员在换了测试服务器（非`dev_server`）后能快速的搭建测试环境而不需要重新搭建

2. 新建一个`ci_server` container


- 新运行一个 container 名为`ci_server`，需要将`ci_server` 的端口22 映射到宿主机
- 安装运行ssh
- 配置好程序编译环境
- 安装git
	1. 创建远端仓库：在`test_repo`目录下执行 `git init --bare`
	2. 将开发机上的需要测试代码分支推到远端仓库后，在 `test_src`目录下执行`git clone $test_repo`
- 配置自动编译部署脚本，在远端仓库中添加 post-recieve hook，hook 包括3部分
	1. pull 刚刚提交的代码
	2. 编译
	3. 部署测试环境
**注意：**
在 hook 中，GIT_DIR 的值被设置为`.`,因此在使用 git 命令前可以删除这个变量，或者将它设置为你需要的的值

## 参考资料
-  [在测试服务器上建立代码的远程仓库&代码仓库](http://www.tuicool.com/articles/3QRB7jU)
- [添加 post_recieve hook](http://git-scm.com/docs/githooks)
- container 中安装 ssh：
```
	$ yum install openssh-server 
	$ echo root:sshpass | chpasswd
	$ sed 's@session\s*required\s*pam_loginuid.so@session optional 	pam_loginuid.so@g' -i /etc/pam.d/sshd
	$ /etc/init.d/sshd start
```

- 将`ci_server`的ssh端口（22）映射的宿主机时，一般无法映射到相同的22端口上，因此开发机的本地仓库添加`ci_server`远端仓库时，需要指定端口号，方法如下： `ssh://git@github.com:22/asdf/asdf.git`