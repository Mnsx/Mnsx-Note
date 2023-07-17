# Git介绍

Git是一个免费的、开源的分布式版本控制系统，可以快速高效的处理从小型到大型的各种项目

Git易于学习，占地面积小，性能极快，它具有廉价的本地库，方便的暂存区域和多个工作流分支等特性。

### 版本控制

版本控制是一种记录文件内容变化，以便来查阅特定版本修订情况的系统

版本控制其实最重要的是可以记录文件修改历史记录，从而让用户能够查看历史版本，方便版本切换

个人开发到团队协作

## 版本控制工具

* 集中式版本控制系统，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连接到这台服务器，取出最新的文件或者提交更新。

*
分布式版本控制系统，客户端提取的不是最新版本的文件快照，而是把代码仓库完整的镜像下来（本地库）。这样任何一处协同工作的文件发生故障，事后都可以用其他客户端的本地仓库进行恢复。因为每个客户端的每一次文件提取操作，实际上都是一次对整个文件仓库的完整备份

## 工作机制

![image-20220419111332125](Picture\git工作机制.png)

## Git和代码托管中心

代码托管中心是基于网路服务器的远程代码仓库，一般我们简单称为远程库

* 局域网
    * GitLab
* 互联网
    * GitHub
    * 码云

# 安装Git

跳过——https://www.bilibili.com/video/BV1vy4y1s7k6?p=7

# Git常用命令

| 命令名称                             | 作用           |
| ------------------------------------ | -------------- |
| git config --global user.name 用户名 | 设置用户签名   |
| git config --global user.email 邮箱  | 设置用户签名   |
| git init                             | 初始化本地库   |
| git status                           | 查看本地库状态 |
| git add 文件名                       | 添加到暂存区   |
| git commit -m "日志文件" 文件名      | 提交到本地库   |
| git reflog                           | 查看历史记录   |
| git reset --hard 版本号              | 版本穿梭       |

## 设置用户签名

基本语法

```git
git config --global user.name 用户名

git config --global user.email 邮箱
```

说明

签名的作用是区分不同操作者身份。用户的签名信息在每一个版本的提交信息中能够看到，以此确定本次提交是谁做的。Git首次安装必须设置一下用户签名

**用户签名和登录Github的账号是没有关系的**

## 初始化本地库

基本语法

```git
git init
```

## 查看本地库状态

基本语法

```git
git status
```

返回结果

```git
On branch master // 目前指向的分支

No commits yet // 提交的内容

Untracked files: // 未被追踪标记的文件
	...
	
nothing added to commit but untracked files present ...
```

## 添加暂存区

基本语法

```git
git add hello.txt
```

## 提交本地库

基本语法

```git
git commit -m "日志信息" 文件名
```

**查看版本信息**

```git
git reflog // 精简版
git log // 完整版
```

## 历史版本

基本语法

```git
git reflog //查看精简版本信息
git log // 查看完整版本信息
```

## 版本穿梭

基本语法

```git
git reset --hard 版本号
```

# Git分支操作

![image-20220419124816253](Picture\公司流程.png)

公司服务运行模式

## 分支介绍

在版本控制过程中，同时推进多个任务，为每个任务，我们就可以创建每个任务的单独分支。使用分支意味着程序员可以把自己的工作从开发主线上分离开来，开发自己分支的时候，不会影响主线分支的运行。（分支的底层就是指针的引用）

## 分支的好处

同时并行多个功能开发，提高开发效率

各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任何影响，失败的分支删除重新开始即可

## 分支操作

| 命令名称          | 作用                         |
| ----------------- | ---------------------------- |
| git branch 分支名 | 创建分支                     |
| git branch -v     | 查看分支                     |
| git checkout 分支 | 切换分支                     |
| git switch 分支   | 切换分支                     |
| git merge 分支    | 把指定的分支合并到当前分支上 |

## 查看分支

基本语法

```git
git branch -v
```

## 创建分支

基本语法

```git
git branch 分支名
```

## 切换分支

基本语法

```git
git checkout 分支名
```

## 合并分支

基本语法

```git
git merge 被融合的分支名
```

> 将hot-fix融合到master中

**冲突产生的原因**

合并分支时，两个分支在同一个文件的同一位置有两套我完全不同的修改。Git无法替我们决定使用哪一个。

创建分支的本质就是多创建一个指针

切换分支的本质就是移动HEAD指针

# Git团队协作机制

## 队内协作

A通过push上传到远程库，B通过clone复制到本地库，B修改完成之后，push到远程库，A就可以通过pull拉去远程库代码

## 跨团队协作

C并非A、B团队人员，通过fork将A团队的代码，加到自己的本地库中，C通过clone将远程库中的代码，拉取到本地库，修改完成后，使用push推到远程库，并且向A团队发送pull-request，当A收到请求后，会进行代码的审核，审核完成后，将代码merge到自己的远程库中，就可以通过pull拉取

# GitHub操作

## 创建远程库

略

## 远程仓库操作

| 命令名称                               | 作用                                                       |
| -------------------------------------- | ---------------------------------------------------------- |
| git remote -v                          | 查看当前所有远程地址别名                                   |
| git remote add 别名 分支地址           | 起别名                                                     |
| git push 别名 分支名                   | 推送本地分支上的内容到远程仓库                             |
| git clone 分支地址                     | 将远程仓库的内容克隆到本地                                 |
| git pull 远程仓库地址别名 远程分支命名 | 将远程仓库对应分支最新版内容拉取下来与当前本地分支直接合并 |

## 创建远程库别名

基本语法

```git
git remote -v
git remote add 别名 分支地址
```

## 推动本地库到远程库

基本语法

```git
git push 别名 分支名
```

## 拉取远程库到本地库

基本语法

```git
git pull 别名 对应分支名
```

## 克隆远程仓库到本地

基本语法

git clone 远程地址

> clone操作 1. 拉取代码 2. 初始化本地仓库 3. 创建别名

## 邀请加入团队

在setting中选择member access 中选择添加，输入B的名称，获取邀请码后，B进入邀请码网址，接受即可

## SSH面密操作

生成SSH方法

```git
ssh-keygen -t rsa -C 邮箱(user.email)
```

生成文件中的进入id_rsa.pub将其中内容复制到远程库setting中的SSH and GPG keys 中

# GitLab

## 简介

GitLab是由GitLabInc开发，使用MIT许可证的基于网络的Git仓库管理工具，且具有wiki和issue跟踪功能。使用Git作为代码管理工具，并在此基础上搭建起来的web服务

https://about.gitlab.cn/install/

## 服务器准备

准备一个系统为CentOS7以上版本的服务器，要求内存4G，磁盘50G

关闭防火墙，并且配置好主机名和ip，保证服务器能够上网

## 安装包准备

官网参考——

1. 从https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm下载rpm文件
2. 使用Xftp将rpm文件传到服务器中
3. 在系统防火墙中打开Http、Https、ssh访问

```
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

4. 确保已经正确的设置DNS，并更改https://gitlab.example.com为访问极狐GitLab实例的URL。安装包将在该URL上自动配置和启动极狐GitLab。建议将极狐GitLab实例的域名以环境变量的形式注入

```
export EXTERNAL_URL=https://gitlab.example.com
```

5. 执行安装命令

```
sudo rpm -Uvh gitlab-jh-14.9.3-jh.0.el7.x86_64.rpm
```

6. 服务初始化

```
gitlab-ctl reconfigure
```

7. 启动服务

```
gitlab-ctl start
```

8. 输入网址进入gitlab，初始化密码
