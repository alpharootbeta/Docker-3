**前提：Centos6.5 镜像，Docker环境**


##### 没有环境参考以下文章：

1.  Docker入门教程:Docker安装

2.  Docker入门教程：创建属于自己的基础镜像


## Step1：上DockerFile代码：

	[root@ZWN2K-5296 python2.7]# ll
	总用量 11524
	-rw-r--r-- 1 root root 673 2月 1 20:42 Dockerfile
	-rw-r--r-- 1 root root 11793433 2月 1 20:42 Python-2.7.3.tar.bz2
	[root@ZWN2K-5296 python2.7]# 
	[root@ZWN2K-5296 python2.7]# cat Dockerfile 
	#####################################
	# Dockerfile build to Python2.7
	# Base for Centos6.5 Images
	#####################################
	# FROM images
	FROM centos6.5:v1.0
	
	# Authe
	MAINTAINER Eycode www.eycode.com
	
	# Yum databases
	RUN	yum clean all && \
		yum -y install \
		gcc* tar bzip2 \
		wget curl 
		
	# Install Python2.7
	ADD	Python-2.7.3.tar.bz2 /usr/src/
	RUN	cd /usr/src/Python-2.7.3 && \
		./configure && \
		make && \
		make install
	RUN	mv /usr/bin/python /usr/bin/python2.6.6 && \
		ln -sf /usr/local/bin/python /usr/bin/python && \
		cp /usr/bin/yum /usr/bin/yum.bak && \
		sed -i 's/\/usr\/bin\/python/\/usr\/bin\/python2.6.6/' /usr/bin/yum
	
	# Set defaults port
	EXPOSE 22
	[root@ZWN2K-5296 python2.7]#
 

> 注意：以上代码不在这里解释，需要查看可以参考eycode 的docker系列文章即可。

 

#### Step2：下载Python2.7地址

	[root@ZWN2K-5296 python2.7]# wget http://python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2

#### Step3：创建Python镜像
	[root@ZWN2K-5296 python2.7]# docker build -t python27-bash:v0.0 .
	[root@ZWN2K-5296 python2.7]# docker images
	REPOSITORY TAG IMAGE ID CREATED VIRTUAL SIZE
	python27-bash v0.0 657949c799d8 24 minutes ago 1.09 GB
	[root@ZWN2K-5296 python2.7]#

#### 一些问题：

	docker build：代表通过Dockerfile文件构建一个新镜像。
	
	-t ：命名该镜像名字
	
	” .  “：DockerFile文件的相对路径

**注意：在dockerbuild中，dockerfile位置的指定必须是使用相对路径**


#### Step4：创建容器

	[root@ZWN2K-5296 python2.7]# docker run -d --name python27 -i python27-bash:v0.0 /bin/bash
	a07bcff31687a19d211f5653db3c572a90404d05fb78241e965d4fca1592ebfa
	[root@ZWN2K-5296 python2.7]# 
	[root@ZWN2K-5296 python2.7]# docker ps -a
	CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                                                           NAMES
	a07bcff31687        python27-bash:v0.0   "/bin/bash"         5 seconds ago       Up 4 seconds        22/tcp                                                          python27                    
	[root@ZWN2K-5296 python2.7]#
 

**注意：生成一个新容器，建议挂载目录，防止容器出现问题而导致实验数据掉失。**


**例如：**

	[root@ZWN2K-5296 dockerfile]# docker run -d -v /python:/python --name python27 -i python27-bash:v0.0 /bin/bash
 

**注意：挂载的目录在宿主机上必须存在，这样才能在容器中生成。**


#### Step5：使用容器

	[root@ZWN2K-5296 python2.7]# docker exec -it python27 /bin/bash
	[root@a07bcff31687 /]# 
	[root@a07bcff31687 /]# python -V
	Python 2.7.3
	[root@a07bcff31687 /]#

**注意：建议使用exec参数进入容器。**


> OK~ python环境有了，eycode就开始去写代码了。


**Docker 容器最大的优点就是可以直接生成想要的环境，不需要重新配置。**


**如果想通过SSH的方式访问容器怎么办？**

> 之后的文章会告诉你怎么做。