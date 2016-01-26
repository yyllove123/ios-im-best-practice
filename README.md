#ios-im-best-practice


探索im最佳实现，比如mqtt，msgpack，以及及时通信ui组件等等

#MQTT

##MQTT介绍
全名为Message Queuing Telemetry Transport，消息队列遥测传输。
MQ Telemetry Transport(MQTT)是一个轻量级的基于发布/订阅的消息协议，它针对与一些不好的网络环境而进行设计,具有开放、简单、轻量级和易于集成的特点。

使用场景：

 - 网络昂贵（中国以流量算钱的世界）、带宽低、网络不可靠。
 - 运行在低端设备上，只有有限的处理能力和内存资源。

协议包括：

 - 发布/订阅的消息模式，提供一对多的消息发布，接触程序间的耦合。
 - 对负载内容屏蔽的消息传输。（？啥意思）
 - 使用 TCP/IP 提供网络连接。
 - 有三种消息发布服务质量：
	 - “至多一次”，消息发布完全依赖底层 TCP/IP 网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。
	 - “至少一次”，确保消息到达，但消息重复可能会发生。
	 - “只有一次”，确保消息到达一次。这一级别可用于如下情况，在计费系统中，消息重复或丢失会导致不正确的结果。
 - 小型传输，开销很小（固定长度的头部是 2 字节），协议交换最小化，以降低网络流量。（2字节，一次能传输多大的数据量？）
 - 使用 Last Will 和 Testament 特性通知有关各方客户端异常中断的机制。（Last Will和Testament 特性是什么？）

###Message格式
一个Message包含了header和body，
header只有两个字节，

第一个字节的8个bit格式如下:

bit数 | 4 bit | 1 bit | 2 bit | 1 bit
---- | ---- | ---- | ---- | ---- |
内容 | Message Type(4 bit) | DUP flag(1 bit) | QoS level(2 bit) | RETAIN

第二个byte包含一个数据包的长度。1个字节可以表示256的长度。

####Message Type
4个bit可以表示16钟不同的类型。

类型         | 枚举 | 描述
---         | --- | --- 
Reserved    | 0 | Reserved(预留类型)
CONNECT     | 1 | Client request to connect to Server(连接服务器所用)
CONNACK     | 2 | 服务器确认包
PUBLISH     | 3 | 发布包
PUBACK      | 4 | 发布确认包
PUBREC      | 5 | 发布包已接收确认包
PUBREL      | 6 | 发布包已处理确认包？
PUBCOMP     | 7 | 发布包接收确认包
SUBSCRIBE   | 8 | 客户端订阅包
SUBACK      | 9 | 订阅包确认包
UNSUBSCRIBE | 10 | 取消订阅
UNSUBACK    | 11 | 取消订阅确认
PINGREQ     | 12 | Ping请求包
PINGRESP    | 13 | Ping相应包
DISCONNECT  | 14 | 断开连接
Reserved    | 15 | 预留类型


####DUP flag
这个值只占1个字节，意义为是否发送副本。默认值为0.

当值为1的时候，需要复制此消息包，发送处理此消息副本，保留原消息包，这个情况一般是用在QoS不为0的情况（必须保证有一次收到消息的情况），包类型一般为为PUBLISH,PUBREL,SUBSCRIBE or UNSUBSCRIBE.变量消息头里需包含Message ID。

（消息ID放哪里的呢？？？？）

####Qos
是网络中定义数据优先级的机制，它定义了一套规则保证网络数据安全

mqtt中定义了3种类型如下：

QoS Value | bit1 | bit2 | Description
--------- | ---- | ---- | -----------
0         | 0    | 0    | 之多一次，发送一次不管有没有收到
1         | 0    | 1    | 至少一次，最少保证一次消息收到，会收到两条的情况
2         | 1    | 0    | 保证一次，只会收到一条消息
3         | 1    | 1    | 预留类型

详情可见[http://baike.baidu.com/view/20897.htm?fr=aladdin](http://baike.baidu.com/view/20897.htm?fr=aladdin)

#### RETAIN
此字段只有一个bit，它只使用在PUBLISH的message中，当客户端发送一个message给服务端，如果这个值为1，则服务端需要持久化这个message，之后会将发布包，发布给当前的订阅者。

当收到一个新的订阅信息，在一个主题上，这个主题之前的retain信息将会发送给订阅者。

*继续看 不太理解*


#### 消息头的第二个byte
这个byte主要用来描述当前message包的数据大小。包括了头信息变量和数据实体（body）。

这个字节的最高位（第8为）一般为0，如果为1，则表示下一个字节也是用来描述长度，最多到4个字节。所以最大可描述的消息长度为111111111,11111111,11111111,01111111,十进制为268 435 455，为256M的大小。

#### 可变头部定义
在消息头的第二个byte开始描述的包内容长度之后，就是可变的头部的内容了，每一种MessageType都有固定的可变头部定义，每个都是不同的解析方式，可变头部之后就是包得内容了。

#### 消息体
消息体主要出现在PUBLISH的包类型下，消息体的样式可以自行定义。自行解析。



##example
[JS Server](https://github.com/mqttjs/MQTT.js)

[Swift CocoaMQTT](https://github.com/emqtt/CocoaMQTT)

## 参考资料
[MQTT协议头部信息 聂勇的博客](http://www.blogjava.net/yongboy/archive/2014/02/15/409893.html)


#MsgPack

##对比
网上大多的文章都拿jsonsmart、messagePack、protobuf作比较，结论是如果数据主要包括字符串，则jsonsmart是王者，如果是数据主要是2进制数据，那messagePack是最佳方案。

我自己总结一下对我来说的各自的优缺点：

- jsonsmart是专为java语言的提供的工具，官网只提供了对java的支持，对其它语言支持力度不足。只能用在服务端和Android等以java为编程语言的场景。
- messagePack确实体现了跨语言的特点，官网基本上支持所有语言，和json非常相像，比较容易理解。
- protobuf是我用过的，它直接操作对象，这样省去了创建对象的时间，确实比较方便。不过，pb文件每次增添属性都需要重新编译生成对应的类文件比较麻烦，而且生成的类阅读性非常的差，要看懂需要一定的实力。官方只支持C++,JAVA,.Net，其它Objective-C的支持，跟不上官网的速度，上次用的时候，编译出来总是需要我手动更改一下里面的一些东西才能用。

以这三个来看的话，还是MessagePack比较中意。

##MsgPack介绍
“msgpack就像是json一样”，确实就像json一样，他就是将json换了一种描述，它将每个基本类型都用一个特殊的字节表示，构造了一种新的json。

他描述的类型有：

空类型：null，nil

bool类型：true 或 false

整型：正整数、负整数、uint8、uint16、uint32、uint64、int8、int16、int32、int64

float类型：float32、float64

str类型：str、str8、str16、str32

bin类型（对象类型）：8字节表示长度、16字节表示长度、32字节表示长度

array类型：2

map类型：

ext类型（扩展类型）：

这样基本可以满足我们的大部分的需求了。

Objcetive-C对象转换成二进制数据传到java端转换成java对象。

##资料
[MsgPack官网](http://msgpack.org) 包可以到这里下载

[MsgPack 官网资料](https://github.com/msgpack/msgpack/blob/master/spec.md)

#通讯UI控件











