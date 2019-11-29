---
title: Linux 环境搭建Android 的 Jenkins 自动构建环境
tags: 新建,模板,小书匠
renderNumberedHeading: true
grammar_cjkRuby: true
---
### 安装 jenkins
使用 wget 直接安装。可在[这里](https://mirrors.tuna.tsinghua.edu.cn/jenkins/debian-stable/?C=M&O=D)查看最新版本

- 下载
 ``` cmd
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/debian-stable/jenkins_2.190.3_all.deb
```
- 安装

```cmd
dpkg -i jenkins_2.190.3_all.deb
```
安装后使用 `dpkg -L jenkins` 命令查看安 jenkins 相关文件如下
![enter description here](./images/1574992181325.png)


配置 webhook 自动构建（需要服务器外网能访问、或者 gitlab 跟 jenkins 都安装在内网）
- 配置
无特殊要求直接使用默认的配置就行，如需要修改配置，打开编辑 default 目录下的 `jenkins` 文件
```cmd
vim /etc/default/jenkins
```
- 启动关闭
启动、关闭、重启分别使用下面三个命令
```cmd
/etc/init.d/jenkins start
/etc/init.d/jenkins stop
/etc/init.d/jenkins restart
```
启动后使用 `/etc/init.d/jenkins status` 查看 jenkins 状态如下则说明启动成功
![/etc/init.d/jenkins status](./images/1574992751847.png)

浏览器打开 "本机 ip+jenkins 配置文件中的端口号（默认 8080）"，即可进入 jenkins 服务器图形界面

# 配置

配置 gitlab 插件

安装参数化构建插件
Build With Parameters 输入框式的参数
Persistent Parameter 下拉框式的参数
Gradle 插件