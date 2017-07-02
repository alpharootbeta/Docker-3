**前提：Docker环境，Centos6.5镜像，CentosLinux，没有的可以找找eycode之前的文章。**

> PS：建议读者可以找谷歌，Docker资料会更加多一点。

 

## 什么是DockerFile？

**DockerFile是Linux命令和Docker自身命令的一个整合文件。这个文件中包括了搭建环境的每一个步骤，Docker根据命令一层层建成一个新镜像。也可以说这个镜像是由层组成。一个DockerFile命令形成镜像的一层。**

 

## DockerFile带来的好处？

1.  DockerFile在占的空间大小来说，根本就不是事，相当于一个脚本文件大小
2.  DockerFile它们简化了从头到尾的流程并极大的简化了部署工作

 

## Dockerfile 语法

> 在我们深入讨论Dockerfile之前，让我们快速过一下Dockerfile的语法和它们的意义。

#### 什么是语法？
非常简单，在编程中，语法意味着一个调用命令，输入参数去让应用执行程序的文法结构。这些语法被规则或明或暗的约束。程序员遵循语法规范以和计算机交互。如果一段程序语法不正确，计算机将无法识别。Dockerfile使用简单的，清楚的和干净的语法结构，极为易于使用。这些语法可以自我释义，支持注释。

 

#### 语法格式：

**Dockerfile语法由两部分构成，注释和命令+参数**


#### No1.   FROM 
	FROM命令可能是最重要的Dockerfile命令。改命令定义了使用哪个基础镜像启动构建流程。
	基础镜像可以为任意镜像。如果基础镜像没有被发现，Docker将试图从Docker image index来查找该镜像。
	FROM命令必须是Dockerfile的首个命令。

 

##### 语法：

	# Usage: FROM [image name]
 

##### 使用：

	FROM ubuntu
**或**

	FROM ubuntu:v1.0

> 解释：左边是镜像名，右边是镜像版本


#### No2.   MAINTAINER
理论上它可以放置于Dockerfile的任意位置。这个命令用于声明作者，并应该放在FROM的后面。

**注意：这个命令比较坑，必须在FROM命令后面，否则build镜像时会直接终止**

##### 语法：

	# Usage: MAINTAINER [name]
 

##### 使用：

	MAINTAINER eycode www.eycode.com

**注意：命令后的说明自定义即可，一般有作者名，邮箱**

 

#### No3.   RUN
RUN命令是Dockerfile执行命令的核心部分。它接受命令作为参数并用于创建镜像。不像CMD命令，RUN命令用于创建镜像（在之前commit的层之上形成新的层）。

 

##### 语法：
	# Usage: RUN [command]
 
##### 使用：

	RUN aptitude install -y riak

**注意：在这个命令后面的参数都可以单独拿出来到Linux系统下运行的命令**

 

#### No4.   ADD
ADD命令有两个参数，源和目标。它的基本作用是从源系统的文件系统上复制文件到目标容器的文件系统。如果源是一个URL，那该URL的内容将被下载并复制到容器中。同时带有解压功能。

##### 语法：

	# Usage: ADD [source directory or URL] [destination directory]
 
##### 普通文件复制使用：

	ADD /my_app_folder /my_app_folder

##### 归档文件复制使用：

	ADD tset.tar.gz /my_app_folder

**注意：ADD遇到归档文件不单单把文件复制到指定位置，ADD机制会自动识别，找到相关的解压工具进行解压操作然后复制文件。当然不是所有的压缩文件都能解压，根据宿主机情况。**

#### No5.   COPY
它的基本作用是从源系统的文件系统上复制文件到目标容器的文件系统

##### 语法：

	# Usage: COPY [source directory ] [destination directory]

##### 使用：

	COPY /my_app_folder /my_app_folder

**注意：ADD与COPY比较，ADD带有复制和解压功能，而COPY单纯只有复制功能，但在效率上说，COPY比ADD复制能力要高。**

#### No6.   EXPOSE
EXPOSE用来指定端口，使容器内的应用可以通过端口和外界交互。

##### 语法：

	# Usage: EXPOSE [port]

##### 使用

	EXPOSE 8080 22 3306

**注意：映射端口可以使用多个，但是在DockerFile上映射端口命令并没有什么用，因为在生成容器时需要重新指定端口，如果没有宿主机会自动分配随机端口使用。**

#### No7.   ENV
用于设置环境变量。这些变量以”key=value”的形式存在，并可以在容器内被脚本或者程序调用。这个机制给在容器中运行应用带来了极大的便利。

##### 语法：

	# Usage: ENV key value
 
##### 使用：

	ENV SERVER_WORKS 4

**注意：DockerFile这个命令比较有意思，相当于变量传递的意思**

#### No8.   CMD
和RUN命令相似，CMD可以用于执行特定的命令。和RUN不同的是，这些命令不是在镜像构建的过程中执行的，而是在用镜像构建容器后被调用。

##### 语法：

	# Usage 1: CMD application "argument", "argument", .. 
	# 在 /bin/sh 中执行，提供给需要交互的应用

	# Usage 2: CMD ["executable","param1","param2"]
	# 使用 exec 执行，推荐方式

	# Usage 3: CMD ["param1","param2"]
	# 提供给 ENTRYPOINT 的默认参数

##### 使用：
	CMD "echo" "Hello docker!"

**注意：CMD命令也很有意思，可以认为是开机启动**

##### Docker开机：
当docker一个镜像正常生成后并不是开机，开机是指通过run指令生成一个容器，开机完成后会返回相关的ID值，这个ID值代表了这个容器的标识。

**一个dockerfile文件中只能执行一条CMD命令，一般会写到文件的最后负责开机启动服务作用，如果文件中存在多条CMD命令，则执行最后的那条。**

#### No9.   ENTRYPOINT
帮助你配置一个容器使之可执行化，如果你结合CMD命令和ENTRYPOINT命令，你可以从CMD命令中移除“application”而仅仅保留参数，参数将传递给ENTRYPOINT命令

##### 语法：

	# Usage: ENTRYPOINT ["executable", "param1", "param2"]
	# 容器启动执行

	# Usage: ENTRYPOINT command param1 param2
	# 在shell中执行

##### 使用：

	ENTRYPOINT echo

**注意：**

作用和CMD一样同样是只执行一条，也是负责开机启动，但最大的区别是CMD在容器启动时可以用其他命令覆盖，而ENTRYPOINT命令却不能被覆盖。
根据官方说法，CMD和ENTRYPOINT只能启动一个服务，但如果是多个服务会怎么办？
 

##### 启用多服务：

	ENTRYPOINT /etc/rc.local

**解释：把所有需要启动的服务写到这个文件中，然后给ENTRYPOINT执行。**

**注意：后来eycode使用发现这种方法成功很低，已经弃用了，推荐使用：supervisor，在后面文章中会介绍**
 

#### No10. ONBUILD
配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

##### 语法：

	# Usage: ONBUILD [INSTRUCTION]

##### 使用：

	ONBUILD ADD . /app/src
	ONBUILD RUN /usr/local/bin/python-build --dir /app/src

OK，基础命令介绍到这里，eycode也是新手，希望老手路过指点一二。


**提供一些Dockerfile文件参考地址：**

[https://github.com/CentOS/CentOS-Dockerfiles](https://github.com/CentOS/CentOS-Dockerfiles "DockerFile参考")
