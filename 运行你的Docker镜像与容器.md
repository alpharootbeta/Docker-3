**前提：你先要有一个镜像和Docker环境，如果都没有点击以下链接**

教你安装Docker：Docker入门教程:Docker安装

教你创建docker镜像：Docker入门教程：创建Docker基础镜像

> 注意：镜像必须要导入了，如果没有导入点击上面链接。

 

## Docker镜像篇
#### Step1：检查镜像

	[root@ZWN2K-5001 ~]# docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
	centos-eycode       latest              11e689052fd0        About a minute ago   362.5 MB
	centos64-bash       latest              4b0f93f40dcc        6 days ago           362.5 MB
	[root@ZWN2K-5001 ~]#
 

#### 一些参数：

	REPOSITORY：对象/镜像名
	
	TAG：当没有指定tag时默认自动填充latest，需要指定时在导入镜像时指定
	
	IMAGE ID：镜像ID，值得注意的是，docker是分层结构的(请注意这一点)
	
	CREATED：启用时间
	
	VIRTUAL SIZE：镜像大小

 

#### Step2：备份镜像

	[root@ZWN2K-5001 ~]# docker save centos-eycode > centos-eycode.tar
	[root@ZWN2K-5001 ~]# ll centos-eycode.tar 
	-rw-r--r-- 1 root root 374601216 1月  26 20:13 centos-eycode.tar
	[root@ZWN2K-5001 ~]#
 

**注意：**

> 将指定镜像保存成 tar 归档文件， docker load 的逆操作。保存后再加载（saved-loaded）的镜像不会丢失提交历史和层，可以回滚。
指定的镜像可以是镜像名也可以是镜像ID
 

#### Step3：删除镜像

	[root@ZWN2K-5001 ~]# docker rmi centos-eycode
	Untagged: centos-eycode:latest
	Deleted: 11e689052fd081ff61fd6e5d7af8446795e7247fbed3577746e59d03ec142194
	[root@ZWN2K-5001 ~]# 
	[root@ZWN2K-5001 ~]# docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
	centos64-bash       latest              4b0f93f40dcc        6 days ago          362.5 MB
	[root@ZWN2K-5001 ~]#
 

#### 一些参数：

	rmi：删除镜像

 

**注意：**

> 如果镜像关系到很多容器，记得先备份，Docker删除之后是很难找回，同样可以是镜像名或镜像ID
镜像在docker以层的方式存在，所以在删除镜像后提示会返回一串值
 

#### 批量删除镜像：

	[root@HKCLOUDN2K--827 docker]# docker rmi $(docker images |awk '{print $3}')
 

#### Step4：导入镜像

	[root@ZWN2K-5001 ~]# docker load < centos-eycode.tar 
	[root@ZWN2K-5001 ~]# docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
	centos-eycode       latest              11e689052fd0        16 minutes ago      362.5 MB
	centos64-bash       latest              4b0f93f40dcc        6 days ago          362.5 MB
	[root@ZWN2K-5001 ~]#
 

**注意：**

**从 tar 镜像归档中载入镜像， docker save 的逆操作。保存后再加载（saved-loaded）的镜像不会丢失提交历史和层，可以回滚。
load参数与import最大的区别就是镜像的完整保存
在新建的基础镜像不能与这种方式导入，详情参考另一篇文章: Docker入门教程：创建Docker基础镜像**
 

## Docker容器篇
#### Step1：创建容器

	[root@ZWN2K-5001 ~]# docker run -d -p 2233:22 -v /eycode:/eycode --name centos6.4 -i centos-eycode /bin/bash
	3f15e48ba18156638c280038319f3228a8a6412b745d6da7cd5768abcd1fdffb
	[root@ZWN2K-5001 ~]#
 

#### 一些参数：

	docker run：启动一个容器，在其中运行指定命令
	
	-d ：后台运行容器，并返回容器ID，如果不加-d参数，启动容器时会卡着，建议使用
	
	-p：添加端口映射，指宿主机指定一个没有使用的端口与容器中的端口进行绑定，当容器给删除后才重新释放端口
	
	-P：添加端口，但是不同的是但宿主机重启或容器重启后端口就会给宿主机释放
	
	2223:22：左边是宿主机端口，右边是容器端口，当不指定宿主机端口，宿主机会随机找一个没有使用的端口进行映射
	
	-v：把宿主机目录挂载到容器中去
	
	–name：自定义容器名，具有唯一性
	
	-i：以交互模式运行容器(可以不写，直接加上镜像名)
	
	/bin/bash：调用容器中的/bin/bash，

**值得注意的是：如果基础镜像没有打包好，虽然镜像正常导入，但是创建不了容器，报错找不到”/bin/bash”文件或目录**
> 这个问题，会在后面文章中描述到底是怎么回事
 

#### Step2：检查容器

	[root@ZWN2K-5001 ~]# docker ps -a
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                                               NAMES
	3f15e48ba181        centos-eycode       "/bin/bash"         6 seconds ago       Up 5 seconds        0.0.0.0:2233->22/tcp                                                centos6.4           
	59761d770cb0        centos64-bash       "/bin/bash"         3 days ago          Up 3 days           0.0.0.0:2221->22/tcp, 0.0.0.0:801->80/tcp, 0.0.0.0:3307->3306/tcp   mysql               
	41565b3fb899        centos64-bash       "/bin/bash"         5 days ago          Up 5 days           0.0.0.0:2244->22/tcp                                                centos6             
	[root@ZWN2K-5001 ~]#
 

#### 一些参数：

	ps -a：列出所有的容器
	
	CONTAINER ID：生成的容器层的汇总值，在出到Dockerfile后会清楚为什么是汇总层值。
	
	IMAGE：该容器基于哪个镜像来生成的
	
	COMMAND：运行的命令
	
	CREATED：运行的时间
	
	STATUS：运行的状态(UP–启动的/EXIT—停止的)
	
	NAMES：镜像的自定义名字
	
	PORTS：被映射的端口

 

#### Step3：进入容器

	[root@ZWN2K-5001 ~]# docker exec -it centos6.4 /bin/bash
	[root@3f15e48ba181 /]# 
	[root@3f15e48ba181 /]# ls
	bin   dev  eycode  lib media  opt   root  selinux  sys  usr
	boot  etc  home    lib64  mnt	 proc  sbin  srv      tmp  var
	[root@3f15e48ba181 /]# 
	[root@3f15e48ba181 /]# ll /eycode/
	total 0
	-rw-r--r-- 1 root root 0 Jan 26 07:30 docker
	[root@3f15e48ba181 /]# 
	[root@3f15e48ba181 /]# exit  #退出容器
 

#### 一些参数：

	exec：进入以运行的容器中
	
	-it：以交互模式进入
	
	exit：退出容器
	
	注意：进入容器还可以另外一个命令：attach
	
	建议使用exec

**注意：在创建容器是挂载的目录也在容器中存在，建议创建容器时挂载目录，这样在删除容器后挂载的目录中的数据依然存在。**

 

#### Step4：备份容器

	[root@ZWN2K-5001 data]# docker ps -a
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                                               NAMES
	3f15e48ba181        centos-eycode       "/bin/bash"         10 minutes ago      Up 10 minutes       0.0.0.0:2233->22/tcp                                                centos6.4           
	59761d770cb0        centos64-bash       "/bin/bash"         3 days ago          Up 3 days           0.0.0.0:2221->22/tcp, 0.0.0.0:801->80/tcp, 0.0.0.0:3307->3306/tcp   mysql               
	41565b3fb899        centos64-bash       "/bin/bash"         5 days ago          Up 5 days           0.0.0.0:2244->22/tcp                                                centos6             
	[root@ZWN2K-5001 data]# 
	[root@ZWN2K-5001 data]# docker export centos6.4 > centos6.4.tar
	[root@ZWN2K-5001 data]# ll centos6.4.tar 
	-rw-r--r-- 1 root root 374599168 1月  26 20:56 centos6.4.tar
	[root@ZWN2K-5001 data]#
 

#### 一些参数：

	export：将指定的容器保存成 tar 归档文件， docker import 的逆操作。
	导出后导入（exported-imported)）的容器会丢失所有的提交历史，无法回滚。
	因此，备份出来的数据很小不像load备份出来的文件很大，但迁移到其他docker上依然可以使用，下文介绍。

#### Step5：删除容器

	[root@ZWN2K-5001 data]# docker ps -a
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                                               NAMES
	3f15e48ba181        centos-eycode       "/bin/bash"         12 minutes ago      Up 12 minutes       0.0.0.0:2233->22/tcp                                                centos6.4           
	59761d770cb0        centos64-bash       "/bin/bash"         3 days ago          Up 3 days           0.0.0.0:2221->22/tcp, 0.0.0.0:801->80/tcp, 0.0.0.0:3307->3306/tcp   mysql               
	41565b3fb899        centos64-bash       "/bin/bash"         5 days ago          Up 5 days           0.0.0.0:2244->22/tcp                                                centos6             
	[root@ZWN2K-5001 data]# docker rm centos6.4
	Error response from daemon: Cannot destroy container centos6.4: Conflict, You cannot remove a running container. Stop the container before attempting removal or use -f
	Error: failed to remove containers: [centos6.4]
	[root@ZWN2K-5001 data]# 
	[root@ZWN2K-5001 data]# docker stop centos6.4
	centos6.4
	[root@ZWN2K-5001 data]# docker rm centos6.4
	centos6.4
	[root@ZWN2K-5001 data]#
 

#### 一些参数：

	rm：这个与删除镜像的少了一个”i” ，用来删除容器

**注意：如果容器已经进入过了，删除容器时会提示先停止，然后删除，如果是新容器是不会有提示的**

 

#### 批量删除容器：

	[root@HKCLOUDN2K--827 docker]# docker rm $(docker ps -a |awk '{print $1}')
**注意：不要用错命令了，查看容器需要结合ps -a 命令获取容器ID值**

 

#### Step6：还原容器

	[root@ZWN2K-5001 data]# cat centos6.4.tar | docker import - centos6.4:v0.0
	8adf9cd7ff65283a418eb8c7a3d857d75ce08661a00ad542d72eaa0f7822657c
	[root@ZWN2K-5001 data]# docker ps -a
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                                                               NAMES
	59761d770cb0        centos64-bash       "/bin/bash"         3 days ago          Up 3 days           0.0.0.0:2221->22/tcp, 0.0.0.0:801->80/tcp, 0.0.0.0:3307->3306/tcp   mysql               
	41565b3fb899        centos64-bash       "/bin/bash"         5 days ago          Up 5 days           0.0.0.0:2244->22/tcp                                                centos6             
	[root@ZWN2K-5001 data]# docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
	centos6.4           v0.0                8adf9cd7ff65        About a minute ago   362.5 MB
	centos-eycode       latest              11e689052fd0        56 minutes ago       362.5 MB
	centos64-bash       latest              4b0f93f40dcc        6 days ago           362.5 MB
	[root@ZWN2K-5001 data]#
 

**注意：备份容器到还原成镜像状态，而不是容器。**

**为什么？**

按照eycod的理解，docker为了实现迁移，备份容器本来就是方便迁移。镜像与容器是密不可分的，容器的基础就是镜像，去掉了镜像，就不会存在容器。假想，如果备份容器，导入的还是容器，那么是不是支撑该容器的镜像也要备份好迁移？答案是不可能的。
根据官方对该参数的解释：”从归档文件（支持远程文件）创建一个镜像， export 的逆操作，可为导入镜像打上标签。导出后导入（exported-imported)）的容器会丢失所有的提交历史，无法回滚。”去掉一些历史数据等等，刚刚好形成了一个新的镜像，同时也印证了之前那篇文章中导入基础镜像不能是用load问题，只能用import参数实现导入

**总结：为了实现多机迁移，容器备份并不是还原成容器，而是镜像。**
 

#### Step7：停止容器

	[root@ZWN2K-5001 data]# docker stop centos6.4
	centos6.4
	[root@ZWN2K-5001 data]#
 

#### Step8：启动容器

	[root@ZWN2K-5001 data]# docker start centos6.4
	centos6.4
	[root@ZWN2K-5001 data]#
 

### 小技巧：如果映射的端口不够用怎么办？删掉容器重新生成吗？不是！添加防火墙规则一样可以

##### 单IP单容器端口扩容

	iptables -t nat -A PREROUTING  -p tcp -m tcp --dport 10050 -j DNAT --to-destination  172.17.0.3:10050
	iptables -t nat -A PREROUTING  -p tcp -m tcp --dport 10051 -j DNAT --to-destination  172.17.0.3:10051
 

##### 单IP多容器端口扩容

	iptables -t nat -A PREROUTING  -p tcp -m tcp --dport 50010 -j DNAT --to-destination  172.17.0.3:10050
	iptables -t nat -A PREROUTING  -p tcp -m tcp --dport 50011 -j DNAT --to-destination  172.17.0.3:10051
 

##### 多IP多容器端口扩容

	iptables -t nat -A PREROUTING -d  10.18.103.2 -p tcp -m tcp --dport 10050 -j DNAT --to-destination 172.17.0.3:10050
	iptables -t nat -A PREROUTING -d  10.18.103.2 -p tcp -m tcp --dport 10051 -j DNAT --to-destination 172.17.0.3:10051
	iptables -t nat -A PREROUTING -d  10.18.103.3 -p tcp -m tcp --dport 10050 -j DNAT --to-destination 172.17.0.4:10050
	iptables -t nat -A PREROUTING -d  10.18.103.3 -p tcp -m tcp --dport 10051 -j DNAT --to-destination 172.17.0.4:10051

> 注意的是：Docker默认的网络环境是通过防火墙的nat表进行转换，docker问题值得探讨。