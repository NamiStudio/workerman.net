# 基本流程
基于Gateway/Worker模型的长链接应用开发者可以直接使用applications/Demo这个应用框架来做。

### applications/Demo这个框架提供了以下基本功能。

1、维持客户端长链接

2、每个客户端链接到服务端后服务端可以将这个链接和客户端uid绑定，当客户端发来消息时可以识别是哪个客户端uid发来的消息

3、在服务端可以调用相关接口给其它客户端发送消息数据

4、提供了定时向客户端发送心跳的功能，用来检测与客户端的联通性

5、客户端可以是任意的，Demo中的客户端是telnet应用程序。即在终端运行```telnet ip 8480```

### ChatDemo的测试方法

  * 终端运行 ```telnet ip 8480``` （ip为WorkerMan运行的服务器ip，本机的话可以使用127.0.0.1）
  * 直接打字回车是向所有人发消息
  * $uid:xxx 是向$uid用户发送消息xxx，类似私聊

## 基于applications/Demo开发流程

### 1、建立一个新的项目

1.1、选定项目名，例如项目名叫ChatRoom，更改```applications/ChatDemo``` 为 ```applications/ChatRoom```

1.2、将配置```workerman/conf/conf.d/Gateway.php```中的 ```worker_file```设置为新的路径```worker_file=../applications/ChatRoom/Bootstrap/Gateway.php```

1.3、将配置```workerman/conf/conf.d/BusinessWorker.conf```中的```worker_file```设置为新的路径```worker_file = ../applications/ChatDemo/Bootstrap/BusinessWorker.php```

1.4、重新启动WorkerMan ```workerman/bin/workermand restart```

1.5 至此，一个新的基于Gateway/Worker模型的项目建立好了

### 2、选定协议

appplications/Demo中使用的默认应用层协议是文本请求末尾加回车，即 Text+"\n",协议解析的文件是Procotols\TextProtocol.php

下面是一个请求数据的样本：```Hi,大家好，我是小明\n```（注意末尾的\n是回车字符，代表一个请求的结束）

TextProtocol协议只是一个例子，开发者可以订制自己的协议，具体可参考上一章WorkerMan基本开发流程中的订制协议部分

### 3、通过接口实现相关业务逻辑
Gateway/Worker模型的业务逻辑入口全部在```applications/ChatRoom/Event.php```中。接口如下：

#### 3.2、Event::onGatewayConnect()
当Gateway进程每收一个客户端链接时触发，如果你的应用需要在此时需要做些操作的话可以在这里实现

#### 3.2、Event::onGatewayMessage($recv_buffer)

此接口就是```Gateway```进程的```dealInput```函数，```Gateway```用这个函数来区分TCP流中的请求边界。根据协议判断请求是否完整，```onGatewayMessage```返回数字N，如果```N=0```，代表```Gateway```进程当前的请求接收完整，紧接着```Gateway```进程会将客户端这个请求转发到```BusinessWorker```处理(```onConnect```或者```onMessage```）。```onGatewayMessage```的实现可以参考基本开发流程章节的实现```dealInput/dealProcess```小节中的```dealInput```部分

#### 3.3、Event::onConnect($recv_buffer)
当客户端发来请求，并且这个客户端对应的socket并没有绑定uid(使用```GateWay::notifyConnectionSuccess($uid)```绑定)时，也就是未验证用户发来的数据都触发这个方法。开发者需要实现这个方法，一般在这个方法中通过客户端传递的$recv_buffer中的数据如用户名密码验证用户是否合法，如果合法得到uid（必须为大于0的数字），在gateway上将uid和对应的socket绑定。则这个socket再次发来请求时会触发```Event::onMessge($uid, $recv_buffer)```，从而直接能获得当前请求是哪个uid发来的。

#### 3.4、Event::onMessage($uid, $recv_message)
当使用```GateWay::notifyConnectionSuccess($uid)```绑定的用户发来消息时触发，$uid为对应socket绑定的uid，用来唯一识别客户端。在```onMessage```里面一般是根据协议解析请求并做处理，如果有需要通过```Gateway::sendToAll/sendToUid```向其它用户发送消息。

#### 3.5、Event::onClose($uid)
当客户端**主动**断开时触发，一般在这里清理用户的数据，例如更新数据库中的在线状态为下线

###  4、与客户端调试
调试除了在程序中打断点，还可以通过```tcpdump```等命令抓取网络的请求来判断网络请求是否整正确。

### 5、发布





