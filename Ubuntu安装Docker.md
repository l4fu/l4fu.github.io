
一个完整的Docker有以下几个部分组成：
dockerClient客户端
Docker Daemon守护进程
Docker Image镜像
DockerContainer容器
本章，我将详细的介绍在Ubuntu-18.04中安装并使用Docker，之前在网上找了很多，总结如下：

```dot
digraph base_flow {
    // 步骤1： 定义digraph的属性
    label = <<B>graphviz使用流程</B>>;
    
   // 步骤2： 定义node、edge的属性
    node[shape=box];

    // 步骤3： 添加node、edge
    graph_attr -> node_edge_attr -> node_edge_added -> custom_attr;

    // 步骤4： 定义特定node，edge的属性
    graph_attr[label="1. 定义digraph的属性"];
    node_edge_attr[label="2. 定义node、edge的属性"];
    node_edge_added[label="3. 添加node、edge"];
    custom_attr[label="4. 定义特定node，edge的属性"];
    }
```
ffff


```puml
@startuml
  Alice -> Bob: Authentication Request
  Bob --> Alice: Authentication Response
  Alice -> Bob: Another atuhentication Request
  Alice <-- Bob: Another authentication Response
@enduml

```
ddd


先更新
`sudo apt update`
![更新](_v_images/430261110233918.png =1024x)
安装依赖
`sudo apt install apt-transport-https ca-certificates curl software-properties-common`
添加Dokcer官方密钥到系统中
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
![准备](_v_images/349641410221144.png =1024x)
添加docker源
`sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"`
这里在更新一下源
`sudo apt update`

安装docker
`sudo apt install docker-ce`
![安装](_v_images/119741610225191.png =1024x)

命令报错及解决
![报错和解决](_v_images/526231610213520.png =1024x)

运行容器
![容器](_v_images/274821710242887.png =1024x)