---
date: 2015-08-16T23:04:24+08:00
isCJKLanguage: "true"
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
## 说明：
利用 docker，git 搭建自动编译部署测试环境，每次 commit 后，能在测试服务器上自动编译并部署部署  
### **步骤：**  
1. 在 `ci_agent`上搭建测试环境  
2. 在 `ci_server` 上创建代码远端仓库 `test_repo`  
3. 在远端仓库中添加 post-recieve hook  
3. 在开发机上本地仓库上新添加一个远程仓库指向的远端仓库  


**注意：**  
`ci_agent`和 `ci_server`都是docker container

## 步骤说明
1. 配置`ci_agent`测试环境：
    + 新建并配置一个 container，部署好程序能正常运行的环境，然后将该 container 保存为image，
    + 创建相关的 Dockfile
    创建 Dockfile 并不是一个必须的步骤，但是创建测试环境的 Dockfile，并保存，能让开发人员在换了测试服务器（非`ci_agent`）后能快速的搭建测试环境而不需要重新搭建

2. 创建`ci_server` container
    + 新运行一个 container 名为`ci_server`，需要将`ci_server` 的端口22 映射到宿主机
    + 安装运行ssh
    + 配置程序编译环境
    + 配置repo
        1. 创建远端仓库：在`test_repo`目录下执行 `git init --bare`
        2. 在开发环境代码仓库中添加一个远端仓库，并将项目的代码推送到test_repo

        2. 在 `test_src`目录下执行`git clone $test_repo`
    + 配置自动编译部署脚本，在远端仓库中添加 post-recieve hook，hook 包括3部分
        1. 刚刚提交的代码 pull 到`test_src`目录
        2. 编译
        3. 部署测试环境

**注意：**
在 hook 中，GIT_DIR 的值被设置为`.`,因此在hook中的脚步使用 git 命令前需要删除这个变量，或者将它设置为你需要的的值

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
