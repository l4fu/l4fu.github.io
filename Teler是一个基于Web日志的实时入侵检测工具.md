# Teler

Teler是一个基于Web日志的实时入侵检测工具，该工具能够帮助广大研究人员实时进行HTTP入侵检测并发出威胁警报，该工具是一个命令行工具，基于社区内的其他多个项目和资源实现其功能。简而言之，Telter是一个快速的基于终端的威胁分析工具，其核心思想就是快速分析和实时搜索威胁！

## 功能介绍

> 实时分析：支持对日志进行实时分析，实时识别可疑活动。
> 
> 警报：当检测到威胁活动时，Teler能够发出警报提醒，并通过Slack、Telegram和Discord等工具来推送消息通知。
> 
> 监控：我们提供了一些威胁指标来帮助大家更加轻松地去监控威胁，这里我们使用的是Prometheus。
> 
> 最新资源：工具内继承的资源会持续更新。
> 
> 灵活的日志格式：Teler支持任意自定义日志格式字符串，具体取决于我们在配置文件中定义的日志格式。
> 
> 增量日志处理：需要数据持久性而不是缓冲流？Teler能够通过磁盘上的持久性选项以增量方式处理日志记录。

## 工具安装

### 二进制代码安装

该工具的安装非常简单，我们可以直接访问该项目的【[Releases页面](https://github.com/kitabisa/teler/releases)】来下载预构建好的二进制代码，然后拆包并运行即可。或者，也可以执行下列命令：

curl -sSfL 'https://ktbs.dev/get-teler.sh' | sh -s -- -b /usr/local/bin

### Docker安装

我们可以使用下列命令来获取Docker镜像：

docker pull kitabisa/teler

### 源码安装

此时需要安装并配置好Go v1.14+环境：

GO111MODULE=on go get -v -u ktbs.dev/teler/cmd/teler

如需更新该工具，可以直接使用go get命令和-u选项。

### GitHub安装

git clone https://github.com/kitabisa/teler

cd teler

make build

mv ./bin/teler /usr/local/bin

## 工具使用

Teler的使用非常简单，直接运行下列命令即可：

\[buffers\] | teler -c /path/to/config/teler.yaml

# 或

teler -i /path/to/access.log -c /path/to/config/teler.yaml

如果使用的是Docker镜像的话，则运行下列命令：

\[buffers\] | docker run -i --rm -e TELER\_CONFIG=/path/to/config/teler.yaml teler

# 或

docker run -i --rm -e TELER\_CONFIG=/path/to/config/teler.yaml teler --input /path/to/access.log

## 工具选项

teler -h

上述命令将显示该工具的帮助信息：

![](_v_images/595512122222693.jpeg)

下面给出的是该工具支持的全部选项：

| <br> | <br> | <br> |
| --- | --- | --- |
| ****选项**** | ****描述**** | ****样例**** |
| \-c,  \--config | Teler配置文件 | kubectl logs nginx | teler -c /path/to/config/teler.yaml |
| \-i,  \--input | 日志分析 | teler -i /var/log/nginx/access.log |
| \-x,  \--concurrent | 设置分析日志的并发级别（默认为20） | tail -f /var/log/nginx/access.log | teler -x 50 |
| \-o,  \--output | 将检测到的威胁存储至文件中 | teler -i /var/log/nginx/access.log -o /tmp/threats.log |
| \--json | 在终端中以JSON格式显示相关的威胁信息 | teler -i /var/log/nginx/access.log --json |
| \--rm-cache | 删除所有缓存资源 | teler --rm-cache |
| \-v,  \--version | 显示当前Teler版本 | teler -v |

## 通知推送

我们提供的通知推送选项如下：

> Slack
> 
> Telegram
> 
> Discord

我们可以按照下列方式配置通知警报：

notifications:

  slack:

    token: "xoxb-..."

    color: "#ffd21a"

    channel: "G30SPKI"

 

  telegram:

    token: "123456:ABC-DEF1234...-..."

    chat\_id: "-111000"

 

  discord:

    token: "NkWkawkawkawkawka.X0xo.n-kmZwA8aWAA"

    color: "16312092"

    channel: "700000000000000..."

我们还可以禁用警报或指定警报发送的地方：

alert:

  active: true

  provider: "slack"

## 工具演示视频

下面给出的是Teler在两种场景下的使用情况：

Buffer-streams：【[点我观看](https://asciinema.org/a/367616)】

Incremental：【[点我观看](https://asciinema.org/a/367610)】

## 工具运行样例

![](_v_images/573662122226939.gif)

## 许可证协议

本项目的开发与发布遵循[Apache](https://github.com/kitabisa/teler/blob/development/LICENSE)开源许可证协议。

## 项目地址

Teler：【[GitHub传送门](https://github.com/kitabisa/teler)】