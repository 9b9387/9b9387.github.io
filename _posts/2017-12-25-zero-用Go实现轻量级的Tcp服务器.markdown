---
layout: post
title: "zero——Go实现轻量级的Tcp服务器"
data: 2017-12-25 14:19:00 +0800
categories: diary
---

[zero](https://github.com/9b9387/zero)是我用Go实现的一个非常轻量的Socket服务器，可用于快速制作游戏Demo，整个项目不超过500行代码。
接收发送数据使用二进制数据流的方式，可以非常方便的配合Protobuf使用。

### Message消息结构
消息结构定义在`message.go`中，收发消息的编码和解码操作在`codec.go`中处理。消息结构定义如下：
```
type Message struct {
	msgSize  int32     // 消息长度
	msgID    int32     // 消息ID
	data     []byte    // 消息数据
	checksum uint32    // 校验码 adler32算法
}
```

### Session会话
每个连接对应一个Session对象，在连接建立的时候创建，并在断开连接的时候删除。

每个Session保存当前的`conn`的指针，可以绑定一个`UserID`。

Session还提供一个key-value map用于保存自定义的信息。

### Connection连接
连接建立时，触发连接事件。`conn.go`实现了接收和发送的方法，接收到消息时使用channel发送到socketservice，并触发`onMessage`事件，所以在处理游戏逻辑的时候数据会是同步的。

每个连接在读取消息时，加入超时检查来实现心跳检查，如果在设置的时间内没有接收到消息，则判断为心跳丢失触发断线事件。

### Socket服务
Socket服务被我封装在`service.go`内。需要注册一下事件，分别处理收到消息，连接，断线：

- RegOnMessageHandler(func(s *zero.Session, msg *zero.Message))
- RegOnConnectHandler(func(s *zero.Session))
- RegOnDisconnectHandler(func(s *zero.Session, err error))

当服务启动时，启动一个协程`acceptHandler`监听连接，直到stopCh接收到数据时，停止服务。

在建立连接后会创建一个新的协程`connectHandler`用于创建`conn`对象和`session`对象。

`conn`创建后会启动负责接收`readCoroutine`和发送`writeCoroutine`的协程。

`session`会被保存在`SocketService.sessions`中进行管理。

**项目地址**：[https://github.com/9b9387/zero](https://github.com/9b9387/zero)
