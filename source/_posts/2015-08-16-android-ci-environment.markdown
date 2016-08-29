---
layout: post
title: "构建Android持续集成环境"
date: 2015-08-16 08:36:32 +0800
comments: true
categories: android, CI, 持续集成
---
本文的主要内容是如何用脚本在服务器上创建一个基于gradle的Android构建环境。
# 目标操作系统
* Ubuntu 14.04
ps: 基本上Linux都适用，不过只在Ubuntu和CentOS上测试过。
<!-- more -->
# 依赖的软件
* Java
* Android SDK

## 安装Oracle JDK
这里我安装的JDK8，我使用的安装方法是相对来说比较绿色的

### 下载Oracle JDK8
	wget --no-cookies --no-check-certificate --header 'Cookie: oraclelicense=accept-securebackup-cookie'  -O 'jdk-8-linux-x64.tgz' 'http://download.oracle.com/otn-pub/java/jdk/8u45-b14/jdk-8u45-linux-x64.tar.gz'

### 安装JDK
	tar zxf /var/tmp/jdk-8-linux-x64.tgz -C /var/tmp
注意：这个包解压出来的根目录叫做 __jdk1.8.0_45__  
注意：鉴于我的使用场景，每一次构建我都需要重建我的环境，因此我在这里把JDK解压到/var/tmp，你可以替换成其他文件夹，下同。
### 如何使用JDK
尚且把这种安装方式叫做绿色安装好了，只需要每次使用Java之前设置JAVA_HOME就可以使用了。  
在上一步中，解压好的JDK的全路径是 __/var/tmp/jdk1.8.0_45__，因此可以这样设置__JAVA_HOME__和__Java__相关的bin

	export JAVA_HOME=/var/tmp/jdk1.8.0_45
	export PATH=${JAVA_HOME}/bin:${PATH}

如果想要一劳永逸的话，需要把上面两个export加到~/.bashrc中或者创建一个profile，如下：

	sudo tee -a /etc/profile.d/jdk1.8.sh << "EOF"
	export JAVA_HOME=/var/tmp/jdk1.8.0_45
	export PATH=${JAVA_HOME}/bin:${PATH}
	EOF

## 安装Android SDK

### 下载及解压
	wget -O /var/tmp/android-sdk.tgz http://dl.google.com/android/android-sdk_r24.2-linux.tgz
	tar zxvf /var/tmp/android-sdk.tgz -C /var/tmp/
	export ANDROID_HOME=/var/tmp/android-sdk-linux

Android SDK也被解压到了/var/tmp，你可以换成其他的目录，但是要注意权限问题。

### 安装需要的SDK和相关工具

	echo y | ${ANDROID_HOME}/tools/android update sdk -s -u -a -t 7,24,139,140

前面的`echo y`是为了在在安装过程中同意Android License。
后面的数字，代表的是各个软件，如：

* 7: Android SDK Build-tools, revision 21.1.2
* 24: SDK Platform Android 5.1.1, API 22, revision 2
* 139: Android Support Repository, revision 15
* 140: Android Support Library, revision 22.2

这些代号需要使用下面的命令查看：

	${ANDROID_HOME}/tools/android list sdk --all

## 64位操作系统
如果是64位的操作系统，还需要执行下面的命令。

	dpkg --add-architecture i386
	apt-get -qqy update
	apt-get -qqy install libncurses5:i386 libstdc++6:i386 zlib1g:i386

到这里基本就可以进行构建了。

## 栗子：在daocloud.io上构建Android应用
```
image: ubuntu:14.04

env:
    - DEPENDENCY_DIR=/var/tmp
    - JAVA_HOME=${DEPENDENCY_DIR}/jdk1.8.0_45
    - ANDROID_HOME=${DEPENDENCY_DIR}/android-sdk-linux
    - PATH=${JAVA_HOME}/bin:${PATH}

install:
    - dpkg --add-architecture i386
    - apt-get -qqy update
    - "apt-get install -y git wget libncurses5:i386 libstdc++6:i386 zlib1g:i386"
    - "wget --no-cookies --no-check-certificate --header 'Cookie: oraclelicense=accept-securebackup-cookie' -O ${DEPENDENCY_DIR}/jdk-8-linux-x64.tgz 'http://download.oracle.com/otn-pub/java/jdk/8u45-b14/jdk-8u45-linux-x64.tar.gz'"
    - tar zxf ${DEPENDENCY_DIR}/jdk-8-linux-x64.tgz -C ${DEPENDENCY_DIR}
    - wget -O ${DEPENDENCY_DIR}/android-sdk.tgz http://dl.google.com/android/android-sdk_r24.2-linux.tgz
    - tar zxvf ${DEPENDENCY_DIR}/android-sdk.tgz -C ${DEPENDENCY_DIR}
    - echo y | ${ANDROID_HOME}/tools/android update sdk -s -u -a -t 7,24,139,140

script:
    - ./gradlew assemble
```
