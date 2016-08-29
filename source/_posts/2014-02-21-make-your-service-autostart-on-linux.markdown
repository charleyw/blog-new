---
layout: post
title: "在CentOS下配置自启动服务"
date: 2014-02-21 16:33:09 +0800
comments: true
categories: 
keywords: linux，init，chkconfig, 自启动, LSB
---
这里主要想介绍一下用`chkconfig`配置自启动服务，包括如何控制服务启动顺序以及启动依赖。虽然本文中提到的脚本的测试环境都是在CentOS上，但是因为使用的配置文件是[**LSB-style**](http://en.wikipedia.org/wiki/Linux_Standard_Base)的，所以在其他Linux上也基本能用，只是使用的工具不一样，如在debian上使用`update-rc.d`。首先从如何简单的添加一个自启动服务说起。
## 如何添加一个自启动服务
1. 首先，你需要一个服务配置脚本，在/etc/init.d下面创建一个文件，如test-init

		touch /etc/init.d/test-init
		chmod +x /etc/init.d/test-init
	给这个文件写入如下内容
<!-- more -->
		#!/bin/sh
			
		# chkconfig: 2345 20 80
		# description: Saves and restores system entropy pool for \
		#              higher quality random number generation.


		case "$1" in
		  start)
				echo "start testinit" >> /root/test.init
		        ;;
		  stop)
				echo "stop testinit" >> /root/test.init
		        ;;
		  restart)
				echo "restart testinit" >> /root/test.init
		        ;;
		  *)
		        echo "Usage: $SCRIPT_NAME {start|stop|restart}" >&2
		        exit 3
		        ;;
		esac
	* 首先是前两行注释的内容，看起来是注释，其实是chkconfig的配置，第一行
	
			chkconfig: 2345 20 80
		

		* 这一行指定了这个脚本在哪几个[Run level](http://en.wikipedia.org/wiki/Runlevel)运行，这里是2345，就是在2345这个几个run level都会启动这个脚本。20 80分别指的启动的优先级和结束的优先级。  
		* 第二行是描述。

	* 接下来的case语句是这个脚本所支持的操作，基本上你至少得支持start和stop，否则启动时会报错，在这个脚本中支持了3个操作：start，stop和restart。

1. 使用chkconfig设置这个脚本自启动：

		chkconfig --add test-init
		
1. 检查一下看成功没有：
	
	* 第一种检查方法：
		
			chkconfig
			
		应该会输出下面的内容：		

		> ...  
		> test-init      	0:off	1:off	2:on	3:on	4:on	5:on	6:off  
		> ...
	
		
	* 第二种：
	
			ls -l /etc/rc3.d/ | grep test-init
			
		应该会输出如下内容
		
		>lrwxrwxrwx  1 root root 19 Feb 21 15:32 S20test-init -> ../init.d/test-init
		
		这里我们只检查了/etc/rc3.d，它存储了run level 3要启动服务的列表。类似的文件夹还有rc0.d, rc1.d, rc2.d, rc4.d, rc5.d, rc6.d，分别对应于相应的run level。这些文件夹里面存储的文件都是对应启动脚本的链接。这些文件的名字遵守固定的规则：
		
		>[S或者K][优先级][服务名称]
		
		系统启动时会按照优先级从小到大执行S开头的脚本的start方法，如对于`S20test-init`就会类似执行这个操作：
		
			/etc/rc3.d/S20test-init start
		
		系统关闭时会按照优先级顺序（方向不确定，有兴趣可以自己去验证一下）执行K开头的脚本的stop方法。

## 如何控制启动顺序
实际上在上面的例子中，我们已经通过chkconfig配置了该服务的优先级，即设定了它在所有服务中的启动顺序：

	chkconfig: 2345 20 80
	
如果我们想让这个服务晚点执行，就可以把启动的优先级改低一点，把20往大了改就可以推迟该服务的启动时间。

当我们有复杂的服务启动依赖关系的时候，比如你的rails应用必须在mysql和邮件服务启动之后才能启动。用上面的这种方法就比较麻烦了。下面介绍另一种设置启动依赖的方法。

### 使用LSB init script设置服务的依赖关系
这种配置文件在linux下是通用的，CentOS，Debian，Ubuntu，OpenSUSE以及RedHat都是支持的。

	### BEGIN INIT INFO
	# Provides:       test-init
	# Required-Start: mysql postfix
	# Required-Stop:  
	# Default-Start:  2 3 4 5
	# Default-Stop:   0 1 6
	# Description:    This is a test script
	### END INIT INFO
	
* **Provides**: 指明这个服务提供那些功能，可以定义多个。实际上可以看做是一些变量，当这个服务启动之后，就可以认为这些变量已经存在了，然后其他服务可以依赖于这些变量的存在性判断是否启动。
* **Required-Start**: 指明这个服务依赖的功能，可以依赖多个。当被依赖的服务出现后，才启动这个服务。不定义任何值的话，就意味着这个脚本可以在系统引导以后直接启动。
* **Required-Stop**: 同上，不过是停止服务。
* **Default-Start**: 在哪些run level启动。
* **Default-Start**: 在哪些run level停止。

把这段配置内容添加到上面脚本的开始位置，再使用chkconfig配置这个脚本：

	chkconfig --del test-init
	chkconfig --add test-init

之后可以使用下面的命令查看一下服务启动的列表：
	
	ls -l /etc/rc3.d/
	
可以发现_**S[优先级]test-init**_排在_**S[优先级]mysql**_和_**S[优先级]postfix**_之后。

## 总结

LSB init script基本上是通用的，在大部分linux上都可以使用。如果不是要启动服务，而是要在启动的时候执行某些命令的话，可以直接写到/etc/rc.local里面，它会在所有服务启动之后执行。