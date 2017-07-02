**致读者：**

1.  如果不想折腾，请使用Centos7.0+以上系统，因为已经集成在系统中，直接使用docker即可
2.  如果想折腾，请一定留意留下的笔记，毕竟坑多，很多朋友读了教程后经常和我抱怨怎么都运行不了，我让他们截图，而这些问题笔记中都是有答案的，我也无奈。有时候结果固然重要，但是过程也是一种学习。


**前提：Centos6.4系统，内核3.10以上，不要听网上不用升级内核就可以正常运行Docker，各种bug各种伤，最后什么也做不到，还换来深深的打击。**


> PS：Centos6.4默认是2.6.32-358.el6.x86_64内核。必须升级，针对公司的产品进行了相关修改，所以可能其他公司的产品，不需要，仅仅提供参考。


**希云的Docker实践视频：中国第一套Docker实战案例视频课程(好像要收费)**

[http://edu.51cto.com/course/course_id-4238.html](http://edu.51cto.com/course/course_id-4238.html "不得不说这是一套入门级最好的教材")

**建议：先学会安装后看视频实践是最好的学习方法**


## 预装环境 ，关闭selinux和开启路由转发功能

	[root@HKCLOUDN2K--827 ~]# vim /etc/selinux/config
	SELINUX=disabled
	[root@HKCLOUDN2K--827 ~]# setenforce 0
	[root@HKCLOUDN2K--827 ~]# vim /etc/sysctl.conf 
	net.ipv4.ip_forward = 1
	[root@HKCLOUDN2K--827 ~]#
 

#### Step1：安装fedora-epel的yum源和hop5源

	[root@HKCLOUDN2K--827 ~]# rpm -ivh http://mirrors.sohu.com/fedora-epel/6/x86_64/epel-release-6-8.noarch.rpm
	[root@HKCLOUDN2K--827 ~]# cd /etc/yum.repos.d/
	[root@HKCLOUDN2K--827 ~]# wget http://www.hop5.in/yum/el6/hop5.repo
 

**注意：**
**必须安装好fedora-epel源，这个是关键源，如果没有安装上，docker是安装后不能正常使用，各种问题。docker软件包不在这个源里，但是支持docker环境的软件包在这个源。
如果提示已经安装过了，但是在/etc/yum.repo.d/目录却没有发现源文件，则卸载后重装**
 

#### 卸载源，重装源
	[root@HKCLOUDN2K--827 ~]# rpm -e epel-release   #卸载源
	[root@HKCLOUDN2K--827 ~]# rpm -ivh http://mirrors.sohu.com/fedora-epel/6/x86_64/epel-release-6-8.noarch.rpm
 

#### Step2：更新CA证书

	[root@HKCLOUDN2K--827 ~]# yum --disablerepo=epel -y update ca-certificates

> 注意：安装好fedora-epel源后，yum是不能正常使用的了，是因为证书错误导致这个问题，eycode一直在纠结这个问题，最后在一篇文章中找到相关方案，完美解决该问题
 

**不过还有一个解决方法，eycode没有试过，读者可以试下**

	sed -i 's/^mirrorlist=https/mirrorlist=http/' /etc/yum.repos.d/epel.repo
 

#### Step3：安装Docker-io
	[root@HKCLOUDN2K--827 ~]# yum clean all && yum makecache
	[root@HKCLOUDN2K--827 ~]# yum -y install lua-alt-getopt lua-filesystem lua-lxc lxc lxc-libs
	[root@HKCLOUDN2K--827 ~]# yum -y install docker-io kernel-ml-aufs
 

**注意：**

 1.  lua-alt-getopt lua-filesystem lua-lxc lxc lxc-libs几个软件包是docker关键环境包，确保一定安装上，如果安装不上请安装docker-io后，重启系统安装。
 2.  安装docker-io包，会自动也安装新内核，内核版本为3.10.5-3.el6.x86_64并支持aufs模块
 3.  如果没有找到kernel-ml-aufs这个软件包，请卸载fedora-epel源和docker-io软件包，检查是否下载了hop5源，然后重装docker-io软件包即可
 

#### Step4：修改启动引导

	[root@HKCLOUDN2K--827 ~]# vim /boot/grub/grub.conf
	#boot=/dev/sda
	default=0    //默认是1，设置为0
	timeout=5
	splashimage=(hd0,0)/grub/splash.xpm.gz
	hiddenmenu
	title CentOS (3.10.5-3.el6.x86_64)
	        root (hd0,0)
	        kernel /vmlinuz-3.10.5-3.el6.x86_64 ro root=UUID=94dcb85c-81c7-45ec-a005-6b895f12961b rd_NO_LUKS  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_MD crashkernel=auto LANG=zh_CN.UTF-8 rd_NO_LVM rd_NO_DM rhgb quiet
	        initrd /initramfs-3.10.5-3.el6.x86_64.img
	title CentOS (2.6.32-358.el6.x86_64)
	        root (hd0,0)
	        kernel /vmlinuz-2.6.32-358.el6.x86_64 ro root=UUID=94dcb85c-81c7-45ec-a005-6b895f12961b rd_NO_LUKS  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_MD crashkernel=auto LANG=zh_CN.UTF-8 rd_NO_LVM rd_NO_DM rhgb quiet
	        initrd /initramfs-2.6.32-358.el6.x86_64.img

**注意：**

1.  修改内核启动项根据读者需要修改而修改，必须确保内核升级到3.10版本以上即可，如果没有找到请返回step3提示操作。
2.  在某些官方教程中还需要挂载/sys/fs/cgroup这个目录，但是经过多次测试，是不需要的，eycode不知道是系统问题还是真的不需要的原因。建议不写，写了启动cgconfig服务是启动失败的，这个慎重选择。

**自动挂载/sys/fs/cgroup目录 [建议不写]**

	[root@HKCLOUDN2K--827 ~]# vim /etc/fstab
	none    /sys/fs/cgroup  cgroup  defaults 0 0

**至于为什么要支持这个请参考 ：Docker入门教程：初涉Docker**

**值得注意的是：在内核3.10版本以下，是不支持挂载这个目录的，必须重启后自动挂载**

 

##### 重启系统…..中……!

 

#### Step5：检查内核/aufs状态

	[root@HKCLOUDN2K--827 ~]# uname -r
	[root@HKCLOUDN2K--827 ~]# grep aufs /proc/filesystems
 

#### Step6：检查Docker状态

	[root@ZWN2K-5296 weida]# ll /etc/sysconfig/docker
	-rw-r--r--. 1 root root 530 8月  15 01:16 /etc/sysconfig/docker
	[root@ZWN2K-5296 weida]# 
	[root@ZWN2K-5296 weida]# docker -h
	Usage: docker [OPTIONS] COMMAND [arg...]
	
	A self-sufficient runtime for linux containers.
	
	Options:
	
	  --api-cors-header=                   Set CORS headers in the remote API
	  -b, --bridge=                        Attach containers to a network bridge
	  --bip=                               Specify network bridge IP
	  -D, --debug=false                    Enable debug mode
	  -d, --daemon=false                   Enable daemon mode
 

#### Step7：启动Docker服务

	[root@ZWN2K-5296 weida]# service docker status
	docker 已死，但 pid 文件仍存
	[root@ZWN2K-5296 weida]# service docker start
	Starting docker:	                                   [确定]
	[root@ZWN2K-5296 weida]#
 

#### OK，Docker入门了