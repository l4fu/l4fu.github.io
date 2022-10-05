# Docker安全实践分享
## 1、序章

随着Docker技术近年来的高速发展和广泛使用，众多互联网公司陆续采用Docker技术作为核心业务的Paas层支撑。那么势必在Docker技术下会给安全技术体系带来新的挑战。

本文章目录索引如下：

> 1、Docker架构简介
> 
> 2、Docker生态
> 
> 3、Docker文件存储驱动
> 
> 4、Docker镜像与层级概念
> 
> 5、Docker namespaces与cgroup概念
> 
> 6、容器Hids的实现
> 
> 7、Docker目前面临的安全问题

## 2、Docker架构简介

Docker 包括三个基本概念：

*   镜像（Image）
*   容器（Container）
*   仓库（Repository）

理解了这三个概念，就理解了 Docker 的整个生命周期。

### 2.1  镜像

  
我们都知道，操作系统分为内核和用户空间。对于 Linux而言，内核启动后，会挂载 root 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个root 文件系统。比如官方镜像ubuntu:18.04就包含了完整的一套 Ubuntu 18.04 最小系统的root文件系统Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。分层存储 ✨因为镜像包含操作系统完整的root 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 \[Union FS\]的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。尝试下docker  pull ubuntu：![](_v_images/241435208229839.jpeg)镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

> FROM debian:stretch  
> RUN apt-get update  
> RUN apt-get install -y gcc libc6-dev make wget  
> RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"  
> RUN mkdir -p /usr/src/redis  
> RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1  
> RUN make -C /usr/src/redis  
> RUN make -C /usr/src/redis install

例如如上Dockerfile 中每一个指令都会建立一层，RUN 也不例外。每一个 RUN 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，commit这一层的修改，构成新的镜像。而上面的这种写法，创建了 7 层镜像。

总结：镜像就是打包的、分层的文件系统。

### 2.2  容器

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 \[命名空间\]因此容器可以拥有自己的root文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID  空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。

总结：容器就是供镜像运行的进程，具有独立的命名空间（namespaces）。

### 2.3  仓库

镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry就是这样的服务。

一个 Docker Registry中可以包含多个 仓库（Repository）；每个仓库可以包含多个 标签（Tag）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签>的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest作为默认标签。

以 \[Ubuntu 镜像\]为例，ubuntu 是仓库的名字，其内包含有不同的版本标签，如，16.04,18.04。我们可以通过ubuntu:16.04，或者ubuntu:18.04来具体指定所需哪个版本的镜像。如果忽略了标签，比如 ubuntu，那将视为 ubuntu:latest。

仓库名经常以 两段式路径 形式出现，比如jwilder/nginx-proxy，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

Docker Registry公开服务：例如Docker Hub\\Quay.io\\Google Container Registry。

私有 Docker Registry：比如Harbor和 Sonatype Nexus。

### 2.4  Docker架构

Docker的文件系统是分层的，它的rootfs在mount之后会转为只读模式。Docker在它上面添加一个新的文件系统，来达成它的只读。

事实上，从下图中，我们能看到多个只读的文件系统，Docker中把他们称为 层（layer）。

是只读的，container部分则是可写的。

如果用户想要修改底层只读层上的文件，这个文件就会被先拷贝到上层，修改后驻留在上层，并屏蔽原有的下层文件。

![1605523038_5fb2565eb1229bfe55f33.png!small?1605523039069](_v_images/237395208228544.jpeg)

最后一部分是容器(container)， 容器是可读写的。

在系统启动时，Docker会首先一层一层的加载，直到最先面的base 。而这些都是只读的。

最后，在最上层添加可读写的一个容器， 这里存放着诸如unique id，网络配置之类的信息。

既然是可读写的，就会有状态。容器共有两种状态：running 和 exited.

用户也可以用Docker commit 命令（不推荐使用）将一个容器压制为，供后续使用。

## 3、Docker生态

### Kubernetes

建于 Docker 之上的Kubernetes可以构建一个容器的调度服务，其目的是让用户透过Kubernetes集群来进行云端容器集群的管理，而无需用户进行复杂的设置工作。系统会自动选取合适的工作节点来执行具体的容器集群调度处理工作。其核心概念是Container Pod。一个Pod由一组工作于同一物理工作节点的容器构成。这些组容器拥有相同的网络命名空间、IP以及存储配额，也可以根据实际情况对每一个 Pod进行端口映射。此外，Kubernetes工作节点会由主系统进行管理，节点包含了能够运行 Docker 容器所用到的服务。

kubelet 是在每个 Node 节点上运行的主要 “节点代理”。它可以通过以下方式向 apiserver 进行注册：主机名（hostname）；覆盖主机名的参数；某云服务商的特定逻辑。

![](_v_images/232355208223505.jpeg)

*   节点（Node）：一个节点是一个运行 Kubernetes 中的主机。
    
*   容器组（Pod）：一个 Pod 对应于由若干容器组成的一个容器组，同个组内的容器共享一个存储卷(volume)。
    
*   容器组生命周期（pos-states）：包含所有容器状态，包括容器组状态类型，容器组生命周期，事件，重启策略，以及 replication controllers。
    
*   Replication Controllers：主要负责指定数量的 pod 在同一时间一起运行。并且在任何时候都有指定数量的Pod副本，在此基础上提供一些高级特性，比如滚动升级和弹性伸缩
    
*   服务（services）：一个 Kubernetes 服务是容器组逻辑的高级抽象，同时也对外提供访问容器组的策略。
    
*   卷（volumes）：一个卷就是一个目录，容器对其有访问权限。
    
*   标签（labels）：标签是用来连接一组对象的，比如容器组。标签可以被用来组织和选择子对象。
    
*   接口权限（accessing\_the\_api）：端口，IP 地址和代理的防火墙规则。
    
*   web 界面（ux）：用户可以通过 web 界面操作 Kubernetes。
    
*   命令行操作（cli）：kubectl命令。
    

## 4、Docker文件存储驱动

Docker的overlay存储驱动利用OverlayFS（堆叠文件系统）的一些特征来构建以及管理镜像和容器的磁盘结构。

docker1.12后推出的overlay2在inode的利用方面比ovelay更有效，overlay2要求内核版本大于等于4.0。

  

### 4.1overlay

### (由较早的kernel以及Docker版本使用)

如下图所示，overlay在主机上用到2个目录，这2个目录被看成是overlay的层。upperdir为容器层、lowerdir为镜像层使用联合挂载技术将它们挂载在同一目录(merged)下，提供统一视图。

![](_v_images/226315208211415.jpeg)

当容器层和镜像层拥有相同的文件时，容器层的文件可见，隐藏了镜像层相同的文件。然后由容器挂载目录(merged)提供了统一视图。

overlay只使用2层，意味着多层镜像不会被实现为多个OverlayFS层。每个镜像被实现为自己的目录，这个目录在路径/var/lib/docker/overlay下。硬链接被用来索引和共享底层的数据，节省了空间。

当创建一个容器时，overlay驱动连接代表镜像层顶层的目录(只读)和一个代表容器层的新目录(读写)：

> root@chenximing-MS-7823:/home/chenximing# tree /var/lib/docker/overlay/
> 
> /var/lib/docker/overlay/

一开始overlay路径下为空，用docker pull命令下载一个由5层镜像层组成的docker镜像:

> chenximing@chenximing-MS-7823:~$ docker pull ubuntu  
> Using default tag: latest  
> latest: Pulling from library/ubuntu  
> c62795f78da9: Pull complete   
> d4fceeeb758e: Pull complete   
> 5c9125a401ae: Pull complete   
> 0062f774e994: Pull complete   
> 6b33fd031fac: Pull complete   
> Digest: sha256:c2bbf50d276508d73dd865cda7b4ee9b5243f2648647d21e3a471dd3cc4209a0  
> Status: Downloaded newer  for ubuntu:latests

此时，在路径/var/lib/docker/overlay/下，每个镜像层都有一个对应的目录，包含了该层镜像的内容，通过tree命令发现，每个镜像层下只包含一个root目录 (因为docker1.10开始，使用基于内容的寻址，因此目录名和镜像层的id不一致)。

> root@chenximing-MS-7823:/home/chenximing# tree -L 2 /var/lib/docker/overlay/
> 
> /var/lib/docker/overlay/  
> ├── 3279a41fd358cdda798d99cc2da0425b4836a489083ae9db4aedc834f426915a  
> │   └── root  
> ├── 33629fe3999e692ecbeb340f855a2d05ddf4e173ea915041be217e1c9cc8a48f  
> │   └── root  
> ├── 6ab876fcc7ad584dd1eebae0d9304abb22561012f6adb87828f57906d799c33b  
> │   └── root  
> ├── 77806a4b8257cd5508a1131d4d47c2d4b4d51703a1cc0dd8daddf0e86a68d492  
> │   └── root  
> └── ed271f2159e3c9de18ede980e4e2ecb0ef39b96927d446ba93deab0b6ff1a8e3  
> └── root

每一层都包含了”该层独有的文件”以及”和其低层共享的数据的硬连接”。如ls命令，每一层都使用一个硬链接来指向实际ls命令的inode号(405241)：

> root@chenximing-MS-7823:/home/chenximing# ls -i /var/lib/docker/overlay/3279a41fd358cdda798d99cc2da0425b4836a489083ae9db4aedc834f426915a/root/bin/ls    405241 /var/lib/docker/overlay/3279a41fd358cdda798d99cc2da0425b4836a489083ae9db4aedc834f426915a/root/bin/ls    root@chenximing-MS-7823:/home/chenximing# ls -i /var/lib/docker/overlay/33629fe3999e692ecbeb340f855a2d05ddf4e173ea915041be217e1c9cc8a48f/root/bin/ls    405241 /var/lib/docker/overlay/33629fe3999e692ecbeb340f855a2d05ddf4e173ea915041be217e1c9cc8a48f/root/bin/ls

使用刚刚拉取的ubuntu镜像创建一个容器（docker run --name test -d -it ubuntu:latest /bin/bash），用docker ps命令查看：

> root@chenximing-MS-7823:/home/chenximing# docker ps  
>   
> CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES  
> 8963ffb422fa        ubuntu              "/bin/bash"         32 minutes ago      Up 52 seconds                           container\_1

创建容器时，实际上是在已有的镜像层上创建了一层容器层，容器层在路径/var/lib/docker/overlay下也存在对应的目录：

> root@chenximing-MS-7823:/home/chenximing# ls /var/lib/docker/overlay/  
>   
> 3279a41fd358cdda798d99cc2da0425b4836a489083ae9db4aedc834f426915a  
> 33629fe3999e692ecbeb340f855a2d05ddf4e173ea915041be217e1c9cc8a48f  
> 6ab876fcc7ad584dd1eebae0d9304abb22561012f6adb87828f57906d799c33b  
> 6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d       //创建容器后新增的目录  
> 6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d-init  //创建容器后新增的目录  
> 77806a4b8257cd5508a1131d4d47c2d4b4d51703a1cc0dd8daddf0e86a68d492  
> ed271f2159e3c9de18ede980e4e2ecb0ef39b96927d446ba93deab0b6ff1a8e3

“6abc47……”和“6abc47…..-init”为创建容器后新增的目录。查看这两个目录的内容：

> root@chenximing-MS-7823:/home/chenximing# tree -L 3 /var/lib/docker/overlay/6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d\*/  
>   
> /var/lib/docker/overlay/6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d/  
> ├── lower-id  
> ├── merged  
> ├── upper  
> │   ├── dev  
> │   │   └── console  
> │   ├── etc  
> │   │   ├── hostname  
> │   │   ├── hosts  
> │   │   ├── mtab -> /proc/mounts  
> │   │   └── resolv.conf  
> │   └── root  
> └── work  
> └── work  
>   
> /var/lib/docker/overlay/6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d-init/  
> ├── lower-id  
> ├── merged  
> ├── upper  
> │   ├── dev  
> │   │   └── console  
> │   └── etc  
> │       ├── hostname  
> │       ├── hosts  
> │       ├── mtab -> /proc/mounts  
> │       └── resolv.conf  
> └── work  
> └── work
> 
>   

“6abc47……”为读写层，“6abc47…..-init”为初始层。初始层中大多是初始化容器环境时，与容器相关的环境信息，如容器主机名，主机host信息以及域名服务文件等。所有对容器做出的改变都记录在读写层。

文件lower-id用来索引该容器使用的镜像层，upper目录包含了容器层的内容，每当启动一个容器时，会将lower-id指向的镜像层目录以及upper目录联合挂载到merged目录，因此，容器内的视角就是merged目录下的内容。而work目录则是用来完成如copy-on\_write的操作。

  

看看容器层使用到了哪一层镜像层：

> root@chenximing-MS-7823:/home/chenximing# cat /var/lib/docker/overlay/6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d/lower-id  
>   
> ed271f2159e3c9de18ede980e4e2ecb0ef39b96927d446ba93deab0b6ff1a8e3  
>   
> root@chenximing-MS-7823:/home/chenximing# cat /var/lib/docker/overlay/6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d-init/lower-id  
>   
> ed271f2159e3c9de18ede980e4e2ecb0ef39b96927d446ba93deab0b6ff1a8e3

在刚才创建的容器中创建一个文件：

> root@8963ffb422fa:/# touch file  
>   
> root@8963ffb422fa:/# echo "hello world" > file  
>   
> root@8963ffb422fa:/# ls  
> bin   dev  file  lib    media  opt   root  sbin  sys  usr  
> boot  etc  home  lib64  mnt    proc  run   srv   tmp  var

此时再观察“6abc47……”目录(读写层)：

> root@chenximing-MS-7823:/home/chenximing# tree -L 3 /var/lib/docker/overlay/6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d/    
>   
> /var/lib/docker/overlay/6abc47fcf668b6820a2a9a8b0d0c6150f9df61ab5a323783630fac3fd282144d/  
> ├── lower-id  
> ├── merged  
> ├── upper  
> │   ├── dev  
> │   │   └── console  
> │   ├── etc  
> │   │   ├── hostname  
> │   │   ├── hosts  
> │   │   ├── mtab -> /proc/mounts  
> │   │   └── resolv.conf  
> │   ├── file  
> │   └── root  
> └── work  
> └── work

发现upper目录下多出了一个file文件，就是刚才在容器中创建的文件。

### 4.2  overlay2

和overlay为了实现“两个目录反映多层镜像“而使用硬链接不同，overlay2驱动天生支持多层。(最多128层)

因此，overlay2在使用docker层相关的命令时，能提供更好的性能(如：docker build、docker commit)。而且overlay2消耗的inode节点更少。

docker pull ubuntu拉取完一个5层的Ubuntu镜像后，/var/lib/docker/overlay2下可以看到几个目录:

![](_v_images/223265208211276.jpeg)

"I"目录包含一些符号链接作为缩短的层标识符. 这些缩短的标识符用来避免挂载时超出页面大小的限制，每个标示符对应不同层级的diff目录。

最底层包含”link”文件(不包含lower文件，因为是最底层)，在上面的结果中“c7fae….”为最底层。这个文件记录着作为标识符的更短的符号链接的名字、最底层还有一个”diff”目录(包含实际内容)。

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]#cat /var/lib/docker/overlay2/c7fae7e34df9196dacbb4a23f858758d14737d9a0e4168056126c7c540d8f6c8/link  
> NAQRQEJKM4Z7EIVPNX3QXGW75U  
>   
> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]#ls /var/lib/docker/overlay2/c7fae7e34df9196dacbb4a23f858758d14737d9a0e4168056126c7c540d8f6c8/diff/  
> bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

从第二层开始，每层镜像层包含”lower“文件，根据这个文件可以索引构建出整个镜像的层次结构。”diff“文件(层的内容)。还包含”merged“和”work”目录，用途和overlay一样。

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# cat /var/lib/docker/overlay2/596c50d705a1fccc2cd6785bdb9fcfd48c5ec72d64e566047184f432747c3eba/link  
> DLZI7ZAHMEL5DPBJPBN2AF7X2Q  
>   
> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# ls /var/lib/docker/overlay2/596c50d705a1fccc2cd6785bdb9fcfd48c5ec72d64e566047184f432747c3eba/diff/  
> run  
>   
> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# cat /var/lib/docker/overlay2/596c50d705a1fccc2cd6785bdb9fcfd48c5ec72d64e566047184f432747c3eba/lower  
> l/IUJFL55XIAQBZV67VRVSWBYVHO:l/NJIGUMOAH4B4VYQUPR3WMO2LRP:l/NAQRQEJKM4Z7EIVPNX3QXGW75U

再来看看容器层，容器层的文件构成和镜像层类似(这点和overlay不同)，使用刚刚拉取的ubuntu镜像创建一个容器(docker run --name test -d ubuntu:latest)，/var/lib/docker/overlay2目录下新增2个目录：

![](_v_images/219225208215025.jpeg)

查看容器层通过lower目录索引的镜像层明细：

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# cat /var/lib/docker/overlay2/2297016015b9a9b359a54b63fccb294977043e502f6063fa25d48590868862fb/lower  
> l/URSCIF77IGWZBPBQJI4Y4GV5T4:l/DLZI7ZAHMEL5DPBJPBN2AF7X2Q:l/IUJFL55XIAQBZV67VRVSWBYVHO:l/NJIGUMOAH4B4VYQUPR3WMO2LRP:l/NAQRQEJKM4Z7EIVPNX3QXGW75U

对于读，考虑下列3种场景：

*   读的文件不在容器层：如果读的文件不在容器层，则从镜像层进行读
    
*   读的文件只存在在容器层：直接从容器层读
    
*   读的文件在容器层和镜像层：读容器层中的文件，因为容器层隐藏了镜像层同名的文件
    

对于写，考虑下列场景：

*   写的文件不在容器层，在镜像层：由于文件不在容器层，因此overlay/overlay2存储驱动使用copy\_up操作从镜像层拷贝文件到容器层，然后将写入的内容写入到文件新的拷贝中
    
*   删除文件和目录：删除镜像层的文件，会在容器层创建一个whiteout文件来隐藏它；删除镜像层的目录，会创建opaque目录，它和whiteout文件有相同的效果
    
*   重命名目录：对一个目录调用rename仅仅在资源和目的地路径都在顶层时才被允许，否则返回EXDEV（场景较少，未做深入研究）
    

总结：

Docker镜像存储存在分层的概念，在不同的版本存储驱动overlay与overlay2中，除了高层级镜像向底层级镜像link文件方式不同外，原理是相同的，每个层级有专属这个层级的文件系统和文件。最后merged目录将镜像层与容器层文件取并集后形成视图。

Docker安全中，webshell等文件检测机制就是通过找到容器文件存储驱动，通过manifest找到容器与宿主机文件驱动的对应关系，再进行文件hash检测与特征码识别即可。

## 5、Docker镜像与层级的概念

### 5.1  Docker镜像与层级详解

![](_v_images/215175208225727.jpeg)

因为镜像包含操作系统完整的root 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 \[Union FS\]的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

最终最高层的镜像层成为只读的init layer层，其与读写权限的容器层以及mount的宿主机文件路径，共同成为容器的文件系统。

尝试下docker  pull ubuntu：

![](_v_images/210125208243514.jpeg)

可以发现一个ubuntu镜像存在多个sha256的digestid，用来标示不同层级的layerid(涂层)，我们可以通过docker inspect ubuntu查看每个镜像层级的digest sha256的值

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# docker inspect ubuntu

![](_v_images/205065208216559.jpeg)

除了基础镜像存储驱动层，还有容器挂载的目录（Volumes）以及dockerfile中指定的WorkDir(工作目录)：

![](_v_images/199025208215950.jpeg)

### 5.2  Docker镜像扫描的实现

之前讲到Docker的存储驱动overlay2，Docker会在/var/lib/docker/目录下给使用的存储驱动创建一个目录：/var/lib/docker/overlay2。

> tree -L 2 overlay2/  
> overlay2/  
> ├── distribution  
> │   ├── diffid-by-digest  
> │   └── v2metadata-by-diffid  
> ├── db  
> │   ├── content  
> │   └── metadata  
> ├── layerdb  
> │   ├── mounts  
> │   ├── sha256  
> │   └── tmp  
> └── repositories.json

这里的关键地方是db和layerdb目录，看这个目录名字，很明显就是专门用来存储元数据的地方，那为什么区分和layer呢？因为在docker中，是由多个layer组合而成的，换句话就是layer是一个共享的层，可能有多个会指向某个layer。

通过docker  ls找到镜像id，我的是1e4467b07108

我们打印/var/lib/docker//overlay2/db/content/sha256这个目录：

> \[root@l-log-analysis-es2.sec.prod.ali.dm sha256\]# ll |grep 1e4467b07108  
> \-rw------- 1 root root 3408 8月  10 15:14 1e4467b07108685c38297025797890f0492c4ec509212e2e4b4822d367fe6bc8

第一行的1e4467b07108685c38297025797890f0492c4ec509212e2e4b4822d367fe6bc8正是记录我们ubuntu镜像元数据的文件，接下来cat一下这个文件，得到一个长长的json：

![](_v_images/194985208223065.jpeg)

由于篇幅原因，我只展示最关键的一部分，也就是rootfs。可以看到rootfs的diff\_ids是一个包含了3个元素的数组，其实这3个元素正是组成ubuntu镜像的3个layerID，从上往下看，就是底层到顶层，也就是说![](_v_images/191935208214118.jpeg)是的最底层。既然得到了组成这个的所有layerID，那么我们就可以带着这些layerID去寻找overlay中对应的文件目录了。

我们返回到上一层的layerdb中，先打印一下这个目录：

> \[root@l-log-analysis-es2.sec.prod.ali.dm layerdb\]# ll  
> 总用量 12  
> drwxr-xr-x 3 root root 4096 8月  10 15:47 mounts  
> drwxr-xr-x 6 root root 4096 8月  10 15:14 sha256  
> drwxr-xr-x 2 root root 4096 8月  10 15:14 tmp

在这里我们只管mounts和sha256两个目录，再打印一下sha256目录:

![](_v_images/188855208211614.jpeg)

/var/lib/docker//overlay2/layerdb存的只是元数据，那么真实的rootfs到底存在哪里呢？其中cache-id就是我们关键所在了。我们打印一下如下目录:

![](_v_images/185775208221084.jpeg)![](_v_images/182735208227950.jpeg)

没错，这个id就是对应/var/lib/docker/overlay2/c7fae7e34df9196dacbb4a23f858758d14737d9a0e4168056126c7c540d8f6c8

通过以上技术手段，那我们能够获取一个镜像()所有layerid对应的overlay2存储驱动的层级，这个层级中diff目录包含了实际容器层使用的文件系统。

> \[root@l-log-analysis-es2.sec.prod.ali.dm layerdb\]# cd /var/lib/docker/overlay2/c7fae7e34df9196dacbb4a23f858758d14737d9a0e4168056126c7c540d8f6c8  
> \[root@l-log-analysis-es2.sec.prod.ali.dm c7fae7e34df9196dacbb4a23f858758d14737d9a0e4168056126c7c540d8f6c8\]# ls  
> diff  link  
> \[root@l-log-analysis-es2.sec.prod.ali.dm c7fae7e34df9196dacbb4a23f858758d14737d9a0e4168056126c7c540d8f6c8\]# cd diff/  
> \[root@l-log-analysis-es2.sec.prod.ali.dm diff\]# ;s  
> bash: 未预期的符号 \`;' 附近有语法错误  
> \[root@l-log-analysis-es2.sec.prod.ali.dm diff\]# ls  
> bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

1、镜像系统版本，安装的软件包版本记录等

> centos：etc/os-release,usr/lib/os-release
> 
>   
> 
>   
> 
> \[root@l-log-analysis-es2.sec.prod.ali.dm diff\]# more usr/lib/os-release  
> NAME="Ubuntu"  
> VERSION="20.04 LTS (Focal Fossa)"  
> ID=ubuntu  
> ID\_LIKE=debian  
> PRETTY\_NAME="Ubuntu 20.04 LTS"  
> VERSION\_ID="20.04"  
> HOME\_URL="https://www.ubuntu.com/"  
> SUPPORT\_URL="https://help.ubuntu.com/"  
> BUG\_REPORT\_URL="https://bugs.launchpad.net/ubuntu/"  
> PRIVACY\_POLICY\_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"  
> VERSION\_CODENAME=focal  
> UBUNTU\_CODENAME=focal

2、探测镜像已安装的软件包

上一步已经探测到了操作系统，自然可以知道系统的软件管理包是rpm还是dpkg。

> debian, ubuntu : dpkg  
> centos, rhel, fedora, amzn, ol, oracle : rpm

centos系统的软件管理包是rpm，而debain系统的软件管理是dpkg。下面是不同软件管理包的目录：

> rpm：var/lib/rpm/Packages  
> dpkg：var/lib/dpkg/status  
> apk：lib/apk/db/installed

比如debian系统，从文件/var/lib/dpkg/status文件则可以探测到当前系统安装了哪些版本的软件。

> \[root@l-log-analysis-es2.sec.prod.ali.dm diff\]# more var/lib/dpkg/status  
> Package: adduser  
> Status: install ok installed  
> Priority: important  
> Section: admin  
> Installed-Size: 624  
> Maintainer: Ubuntu Core Developers <ubuntu-devel-discuss@lists.ubuntu.com>  
> Architecture: all  
> Multi-Arch: foreign  
> Version: 3.118ubuntu2  
> Depends: passwd, debconf (>= 0.5) | debconf-2.0  
> Suggests: liblocale-gettext-perl, perl, ecryptfs-utils (>= 67-1)  
> Conffiles:  
> /etc/deluser.conf 773fb95e98a27947de4a95abb3d3f2a2  
> Description: add and remove users and groups  
> This package includes the 'adduser' and 'deluser' commands for creating  
> and removing users.  
> .  
> \- 'adduser' creates new users and groups and adds existing users to  
> existing groups;  
> \- 'deluser' removes users and groups and removes users from a given  
> group.  
> .  
> Adding users with 'adduser' is much easier than adding them manually.  
> Adduser will choose appropriate UID and GID values, create a home  
> directory, copy skeletal user configuration, and automate setting  
> initial values for the user's password, real name and so on.  
> .  
> Deluser can back up and remove users' home directories  
> and mail spool or all the files they own on the system.  
> .  
> A custom script can be executed after each of the commands.  
> Original-Maintainer: Debian Adduser Developers <adduser@packages.debian.org>
> 
> Package: apt  
> Status: install ok installed  
> Priority: important  
> Section: admin  
> Installed-Size: 4182  
> Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>  
> Architecture: amd64  
> Version: 2.0.2ubuntu0.1  
> Replaces: apt-transport-https (<< 1.5~alpha4~), apt-utils (<< 1.3~exp2~)  
> Provides: apt-transport-https (= 2.0.2ubuntu0.1)  
> Depends: adduser, gpgv | gpgv2 | gpgv1, libapt-pkg6.0 (>= 2.0.2ubuntu0.1), ubuntu-keyring, libc6 (>= 2.15), libgcc-s1 (>= 3.0), libgnutls30 (>= 3.6.12), libseccomp2 (>= 2.4.2), libstdc++6 (>= 9), libsystemd0  
> Recommends: ca-certificates  
> Suggests: apt-doc, aptitude | synaptic | wajig, dpkg-dev (>= 1.17.2), gnupg | gnupg2 | gnupg1, powermgmt-base  
> Breaks: apt-transport-https (<< 1.5~alpha4~), apt-utils (<< 1.3~exp2~), aptitude (<< 0.8.10)  
> Conffiles:  
> /etc/apt/apt.conf.d/01-vendor-ubuntu c69ce53f5f0755e5ac4441702e820505  
> /etc/apt/apt.conf.d/01autoremove ab6540f7278a05a4b7f9e58afcaa5f46  
> /etc/cron.daily/apt-compat 49e9b2cfa17849700d4db735d04244f3  
> /etc/kernel/postinst.d/apt-auto-removal 4ad976a68f045517cf4696cec7b8aa3a  
> /etc/logrotate.d/apt 179f2ed4f85cbaca12fa3d69c2a4a1c3  
> Description: commandline package manager  
> This package provides commandline tools for searching and  
> managing as well as querying information about packages  
> as a low-level access to all features of the libapt-pkg library.

现在horbor已经集成Clair镜像安全扫描组件：https://blog.csdn.net/arnolan/article/details/102666831?utm\_medium=distribute.pc\_relevant.none-task-blog-baidujs-1

总结：

镜像使用的Union FS具备分层结构（Layer），分为镜像层+容器读写层+挂载Volume，而这些层级在宿主机是依靠上一章节讲述的Overlay技术进行驱动的，那么这些层级在宿主机是有1对1映射的目录的。然后不同类型的OS软件安装包是有固定目录的，获取到dpkg或是rpm包安装信息，再去匹配该版本软件包CVE漏洞即可。

## 6、Docker namespaces与cgroup概念

### 6.1  namespaces

命名空间 (namespaces) 是 Linux 为我们提供的用于分离进程树、网络接口、挂载点以及进程间通信等资源的方法。在日常使用 Linux 或者 macOS 时，我们并没有运行多个完全分离的服务器的需要，但是如果我们在服务器上启动了多个服务，这些服务其实会相互影响的，每一个服务都能看到其他服务的进程，也可以访问宿主机器上的任意文件，这是很多时候我们都不愿意看到的，我们更希望运行在同一台机器上的不同服务能做到完全隔离，就像运行在多台不同的机器上一样。Docker 就通过 Linux 的 namespaces 对不同的容器实现了隔离。

Linux 的命名空间机制提供了以下七种不同的命名空间，其中包括

CLONE\_NEWCGROUP：Docker未使用。

CLONE\_NEWIPC：IPC namespaces，用于隔离进程间通讯所需的资源。

CLONE\_NEWNET：Network namespaces，为进程提供了一个完全独立的网络协议栈的视图。

CLONE\_NEWNS：mount namespaces，每个进程都存在于一个mount namespaces里面，mount namespaces为进程提供了一个文件层次视图。

CLONE\_NEWPID：PID namespaces，linux通过命名空间管理进程号，同一个进程，在不同的命名空间进程号不同。

CLONE\_NEWUSER：user namespaces，用于隔离用户。

CLONE\_NEWUTS：UTS namespaces，用于隔离主机名。

通过这七个选项我们能在创建新的进程时设置新进程应该在哪些资源上与宿主机器进行隔离。

### 6.1.1  进程namespaces

我们在当前的 Linux 操作系统下运行一个新的 Docker 容器，并通过 exec进入其内部的 bash并打印其中的全部进程，我们会得到以下的结果：

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# docker run --name mytest -d  -it ubuntu:latest  
> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# docker exec -it 15e93faf323c /bin/bash  
> root@15e93faf323c:/#  
> root@15e93faf323c:/#  
> root@15e93faf323c:/# ps -ef  
> UID        PID  PPID  C STIME TTY          TIME CMD  
> root         1     0  0 08:20 pts/0    00:00:00 /bin/bash  
> root        10     0  0 08:21 pts/1    00:00:00 /bin/bash  
> root        18    10  0 08:22 pts/1    00:00:00 ps -ef

在新的容器内部执行ps命令打印出了非常干净的进程列表，只有包含当前ps -ef在内的三个进程，在宿主机器上的几十个进程都已经消失不见了。

当前的 Docker 容器成功将容器内的进程与宿主机器中的进程隔离，如果我们在宿主机器上打印当前的全部进程时，会得到下面三条与 Docker 相关的结果：

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# ps -ef|grep docker  
> root     10229 11636  0 16:20 ?        00:00:00 docker-containerd-shim -namespace moby -workdir /var/lib/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/15e93faf323c5bfbc0897f09f0f3fd371d149198d96bf16ca487d971fe584db2 -address /var/run/docker/containerd/docker-containerd.sock -containerd-binary /usr/bin/docker-containerd -runtime-root /var/run/docker/runtime-runc  
> root     11092  6474  0 16:23 pts/0    00:00:00 grep --color=auto docker  
> root     11625     1  0 8月05 ?       00:13:20 /usr/bin/dockerd  
> root     11636 11625  0 8月05 ?       00:46:11 docker-containerd --config /var/run/docker/containerd/containerd.toml

而当我们在宿主机中，使用pstree -p查看进程树：

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# pstree -p

发现docker中bash进程PID为10271:

> ├─dockerd(11625)─┬─docker-containe(11636)─┬─docker-containe(10229)─┬─bash(10271)  
> │                │                        │                        ├─{docker-containe}(10232)  
> │                │                        │                        ├─{docker-containe}(10233)  
> │                │                        │                        ├─{docker-containe}(10234)  
> │                │                        │                        ├─{docker-containe}(10235)  
> │                │                        │                        ├─{docker-containe}(10240)  
> │                │                        │                        ├─{docker-containe}(10245)  
> │                │                        │                        ├─{docker-containe}(10249)  
> │                │                        │                        └─{docker-containe}(10568)  
> │                │                        ├─{docker-containe}(11637)

通过lsof查看10271进程关联的文件:

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# lsof -p 10271  
> COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF     NODE NAME  
> bash    10271 root  cwd    DIR   0,38     4096 51640972 /  
> bash    10271 root  rtd    DIR   0,38     4096 51640972 /  
> bash    10271 root  txt    REG  253,1  1183448  1573083 /usr/bin/bash  
> bash    10271 root  mem    REG  253,1           1573866 /usr/lib/x86\_64-linux-gnu/libnss\_files-2.31.so (stat: No such file or directory)  
> bash    10271 root  mem    REG  253,1           1573801 /usr/lib/x86\_64-linux-gnu/libc-2.31.so (stat: No such file or directory)  
> bash    10271 root  mem    REG  253,1           1573812 /usr/lib/x86\_64-linux-gnu/libdl-2.31.so (stat: No such file or directory)  
> bash    10271 root  mem    REG  253,1           1573921 /usr/lib/x86\_64-linux-gnu/libtinfo.so.6.2 (stat: No such file or directory)  
> bash    10271 root  mem    REG  253,1           1573779 /usr/lib/x86\_64-linux-gnu/ld-2.31.so (stat: No such file or directory)  
> bash    10271 root    0u   CHR  136,0      0t0        3 /dev/pts/0  
> bash    10271 root    1u   CHR  136,0      0t0        3 /dev/pts/0  
> bash    10271 root    2u   CHR  136,0      0t0        3 /dev/pts/0  
> bash    10271 root  255u   CHR  136,0      0t0        3 /dev/pts/0

由此可见在当前的宿主机器上，可能就存在由上述的不同进程构成的进程树：

![](_v_images/179695208234404.jpeg)

这就是在使用clone创建新进程时传入CLONE\_NEWPID实现的，也就是使用 Linux 的命名空间实现进程的隔离，Docker 容器内部的任意进程都对宿主机器的进程一无所知。

下面我们反向举例，通过将命名空间更改为主机，容器还可以查看系统上运行的所有其他进程。失去了namespaces的隔离作用，容器与宿主机出现了安全隐患。

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# docker run -it --pid=host ubuntu ps aux  
> USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND  
> root         1  0.0  0.0  43252  3520 ?        Ss   Jul20   4:20 /usr/lib/system  
> root         2  0.0  0.0      0     0 ?        S    Jul20   0:00 \[kthreadd\]  
> root         3  0.0  0.0      0     0 ?        S    Jul20   0:00 \[ksoftirqd/0\]  
> root         5  0.0  0.0      0     0 ?        S<   Jul20   0:00 \[kworker/0:0H\]  
> root         7  0.0  0.0      0     0 ?        S    Jul20   0:01 \[migration/0\]  
> root         8  0.0  0.0      0     0 ?        S    Jul20   0:00 \[rcu\_bh\]  
> root         9  0.0  0.0      0     0 ?        S    Jul20   5:12 \[rcu\_sched\]  
> root        10  0.0  0.0      0     0 ?        S    Jul20   0:04 \[watchdog/0\]  
> root        11  0.0  0.0      0     0 ?        S    Jul20   0:03 \[watchdog/1\]  
> root        12  0.0  0.0      0     0 ?        S    Jul20   0:09 \[migration/1\]  
> root        13  0.0  0.0      0     0 ?        S    Jul20   0:03 \[ksoftirqd/1\]  
> root        15  0.0  0.0      0     0 ?        S<   Jul20   0:00 \[kworker/1:0H\]  
> root        16  0.0  0.0      0     0 ?        S    Jul20   0:03 \[watchdog/2\]  
> root        17  0.0  0.0      0     0 ?        S    Jul20   0:00 \[migration/2\]  
> root        18  0.0  0.0      0     0 ?        S    Jul20   0:01 \[ksoftirqd/2\]  
> root        20  0.0  0.0      0     0 ?        S<   Jul20   0:00 \[kworker/2:0H\]  
> root        21  0.0  0.0      0     0 ?        S    Jul20   0:03 \[watchdog/3\]  
> root        22  0.0  0.0      0     0 ?        S    Jul20   0:09 \[migration/3\]  
> root        23  0.0  0.0      0     0 ?        S    Jul20   0:00 \[ksoftirqd/3\]  
> root        25  0.0  0.0      0     0 ?        S<   Jul20   0:00 \[kworker/3:0H\]

总结：

即对于业务进程而已，Docker上的进程其实对于主机上是可见的，因为Docker的namespaces一般都是继承自主机的root namespaces。我们可以通过直接扫描主机上的进程行为来识别当前docker中运行的进程是否有问题。

### 6.1.2  网络namespaces

如果 Docker 的容器通过 Linux 的命名空间完成了与宿主机进程的网络隔离，但是却又没有办法通过宿主机的网络与整个互联网相连，就会产生很多限制，所以 Docker 虽然可以通过命名空间创建一个隔离的网络环境，但是 Docker 中的服务仍然需要与外界相连才能发挥作用。

每一个使用docker run启动的容器其实都具有单独的网络命名空间，Docker 为我们提供了四种不同的网络模式，Host（透明模式，和宿主机共享namespaces）、Container(与其他容器共享namespaces)、None(有独立的namespaces，但没有分配veth和ip) 和 Bridge（网桥模式） 模式。

![](_v_images/175605208239268.jpeg)

在这一部分，我们将介绍 Docker 默认的网络设置模式：网桥模式。在这种模式下，除了分配隔离的网络命名空间之外，Docker 还会为所有的容器设置 IP 地址。当 Docker 服务器在主机上启动之后会创建新的虚拟网桥 docker0，随后在该主机上启动的全部服务在默认情况下都与该网桥相连。

![](_v_images/172545208216828.jpeg)在默认情况下，每一个容器在创建时都会创建一对虚拟网卡，两个虚拟网卡组成了数据的通道，其中一个会放在创建的容器中，会加入到名为 docker0 网桥中。我们可以使用如下的命令来查看当前网桥的接口：

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# brctl show  
> bridge name bridge id  STP enabled interfaces  
> docker0  8000.024280d3380b no  veth2228614

docker0 会为每一个容器分配一个新的 IP 地址并将 docker0 的 IP 地址设置为默认的网关。网桥 docker0 通过 iptables 中的配置与宿主机器上的网卡相连，所有符合条件的请求都会通过 iptables 转发到 docker0 并由网桥分发给对应的机器。

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# iptables -t nat -L  
> Chain PREROUTING (policy ACCEPT)  
> target     prot opt source               destination  
> DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL
> 
> Chain INPUT (policy ACCEPT)  
> target     prot opt source               destination
> 
> Chain OUTPUT (policy ACCEPT)  
> target     prot opt source               destination  
> DOCKER     all  --  anywhere            !loopback/8           ADDRTYPE match dst-type LOCAL
> 
> Chain POSTROUTING (policy ACCEPT)  
> target     prot opt source               destination  
> MASQUERADE  all  --  172.17.0.0/16        anywhere
> 
> Chain DOCKER (2 references)  
> target     prot opt source               destination  
> RETURN     all  --  anywhere             anywhere

我们在当前的机器上使用docker run -d -p 6379:6379 redis命令启动了一个新的 Redis 容器，在这之后我们再查看当前iptables的 NAT 配置就会看到在DOCKER的链中出现了一条新的规则：

> DNAT       tcp  --  anywhere             anywhere             tcp dpt:6379 to:192.168.0.4:6379

上述规则会将从任意源发送到当前机器 6379 端口的 TCP 包转发到 192.168.0.4:6379 所在的地址上。

我们就可以推测出 Docker 是如何将容器的内部的端口暴露出来并对数据包进行转发的了；当有 Docker 的容器需要将服务暴露给宿主机器，就会为容器分配一个 IP 地址，同时向 iptables 中追加一条新的规则。

### 6.1.3  挂载点namespaces

如果一个容器需要启动，那么它一定需要提供一个根文件系统（rootfs），容器需要使用这个文件系统来创建一个新的进程，所有二进制的执行都必须在这个根文件系统中。

![](_v_images/169515208235587.jpeg)想要正常启动一个容器就需要在 rootfs 中挂载以上的几个特定的目录，除了上述的几个目录需要挂载之外我们还需要建立一些符号链接保证系统 IO 不会出现问题。比如/dev目录下的特殊设备，标准输入与错误输出等。

![](_v_images/167475208238085.jpeg)

如果需要对挂载（mount）进行namespaces控制，那么就需要在新的容器进程中创建隔离的挂载点命名空间需要在 clone函数中传入 CLONE\_NEWNS，这样子进程就能得到父进程挂载点的拷贝，如果不传入这个参数子进程对文件系统的读写都会同步回父进程以及整个主机的文件系统。

举个例子：

在没声明挂载命名空间的情况下，且docker启动用户为root用户的情况下，启动docker容器删除宿主机/bin目录下文件。

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# sudo cp /bin/touch /bin/touch.bak

> \[root@l-log-analysis-es2.sec.prod.ali.dm lijianxin1\]# docker run -it -v /bin/:/host/ alpine rm -f /host/touch.bak

> \[root@l-log-analysis-es2.sec.prod.ali.dm bin\]# ls |grep touch

### 6.2  cgroup

我们通过 Linux 的命名空间为新创建的进程隔离了文件系统、网络并与宿主机器之间的进程相互隔离，但是命名空间并不能够为我们提供物理资源上的隔离，比如 CPU 或者内存，如果在同一台机器上运行了多个对彼此以及宿主机器一无所知的『容器』，这些容器却共同占用了宿主机器的物理资源。

![](_v_images/164375208240481.jpeg)

如果其中的某一个容器正在执行 CPU 密集型的任务，那么就会影响其他容器中任务的性能与执行效率，导致多个容器相互影响并且抢占资源。如何对多个容器的资源使用进行限制就成了解决进程虚拟资源隔离之后的主要问题，而 Control Groups（简称 CGroups）就是能够隔离宿主机器上的物理资源，例如 CPU、内存、磁盘 I/O 和网络带宽。

> \--cpu-shares #限制cpu共享  
> \--cpuset-cpus #指定cpu占用  
> \--memory-reservation #指定保留内存  
> \--kernel-memory #内核占用内存  
> \--blkio-weight (block IO) #blkio权重  
> \--device-read-iops #设备读iops  
> \--device-write-iops #设备写iops

## 7、容器Hids的实现

主机入侵检测系统通常情况下是按在主机上的agent对主机上的文件（包括可执行文件，配置文件，脚本文件等），进程，网络情况，日志审计情况等进行扫描和恶意识别，并且及时预警主机的安全状况。

其实主机上单独安装agent是可以完成胜任Docker的扫描和审计的。Docker的可读写层文件系统实际上是会挂载主机上的文件系统，可以通过从主机上来的相关文件目录扫描来识别Webshell，恶意二进制文件等。对于业务进程而已，Docker上的进程其实对于主机上是可见的，因为Docker的namespaces一般都是继承自主机的root namespaces。可以通过直接扫描主机上的进程行为来识别当前Docker中运行的进程是否有问题。

感兴趣的同学可以参考适用于Docker的开源入侵检测系统Sysdig Falco。

## 8、Docker目前面临的安全问题

![](_v_images/161235208222601.jpeg)

主要分为几方面：

1\. Docker自身漏洞

***

代码执行、权限提升、信息泄漏、runc

2\. Docker生态问题

***

K8S漏洞

3\. Docker源问题

***

恶意镜像、存在漏洞的镜像

4\. Docker架构缺陷与安全机制

namespaces导致的：容器之间的局域网攻击、共享root、未隔离的文件系统、默认放通所有

cgroup导致的：DDoS攻击耗尽资源

5\. Docker安全基线

***

内核级别的：namespaces、cgroup、SElinux

主机级别的：服务最小化、禁止挂载宿主机敏感目录、挂载目录权限设置为644

网络级别的：禁止映射特权端口、通过iptable设定规则并禁止容器之间的网络流量

镜像级别的：创建本地镜像服务器、使用可信镜像、使用镜像扫描、合理管理镜像标签

容器级别的：容器以单一主进程方式运行、禁止运行ssh等高危服务、以只读方式挂载根目录系统

6\. Docker安全规则

***

Docker remote api访问控制

使用普通用户启动Docker服务

Docker client与Docker Daemon通讯安全

  

## 9、引用&致谢

感谢引用中各位大佬和团队的分享精神，如果文章中有任何描述不正确或引用不当的地方，辛苦大佬们指正。

也欢迎各位安全同僚拍砖、交流与技术分享！（微信：nixinge66）

转载请标明出处，谢谢各位大佬！

引用来源：

> 1、Docker安全入门与实战（一）https://zhuanlan.zhihu.com/p/43586159
> 
> 2、Docker存储驱动—Overlay/Overlay2「译」https://arkingc.github.io/2017/05/05/2017-05-05-docker-filesystem-overlay
> 
> 3、Docker 核心技术与实现原理   https://draveness.me/docker/