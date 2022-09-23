# 野火IM 推送服务
作为野火IM的推送服务的演示，支持小米、华为、魅族、OPPO、Vivo、苹果apns和谷歌FCM。并且可以添加更多的推送厂商和自定义推送模式。

## 工作原理
推送功能对于所有IM来说都是非常重要的功能，然而手机系统又没有统一的推送服务，对接起来难度很大。另外一方面客户有不同对接需求，有的要求使用第三方，有的要求使用厂商推送，有的需要在海外添加谷歌推送，有的对推送的格式有不同的要求。

为了满足各种各样的需求，提供足够好的灵活性，野火IM把推送子系统独立出来，客户只要理解了推送子系统运行的原理，就能做好各种自定义处理。
![架构图](https://docs.wildfirechat.cn/architecture/wildfire_architecture.png)
> 如果架构图无法查看，可以点击[这里](https://docs.wildfirechat.cn/architecture/wildfire_architecture.png)查看

图中紫色部分为推送子系统，推送子系统的所有源码都是开源的，且可以随意修改。推送子系统的工作流程如下：
1. 应用启动后，推送SDK初始化，判断采用那种推送服务，比如华为手机就用华为推送，小米手机就用小米推送，或者全部或部分使用第三方推送。如果客户要加其它推送也是在这里加。选定好推送厂商后，就初始化对应推送厂商的SDK，注册成功后会得到推送token，调用IM SDK的setDeviceToken，传入推送token和类型。注意类型是可以扩展的，而且对IM系统没有任何影响的。
2. SDK被调用setDeviceToken后，会把推送token和类型传入到IM服务，IM服务为对应手机保存下来以备后用。事实上IM服务不需要理解token和type的含义，只需要透传给推送服务即可。
3. IM服务处理消息时发现用户不在线或者下发消息失败，则会启动是否要推送的决策，比如消息是否需要推送（预制消息已经支持，自定义消息需要传入push content），用户是否全局静音，会话是否被静音，客户有多少天没有登录（超过7天没登录就不推送）。达到推送条件后，跟把所有推送需要的内容打包发给推送服务。
4. 推送服务接收到IM服务的请求，把推送数据放到消费队列中并立即返回（IM服务不能被阻塞），然后逐步处理推送事件。每个推送事件中都包含了所有需要处理的数据，其中包括1步骤中的推送Token和类型，然后根据类型来调用对用推送厂商的服务，比如华为/小米/苹果/第三方厂商/谷歌/OPPO/Vivo等，调用他们的SDK进行推送。
5. 系统厂商或第三方推送厂商利用他们的通道推送到客户端，一般有2种形式：一种是通知栏，不激活应用只弹出通知栏；另外一种形式是透传，把应用激活并把数据传递到客户端的推送相关代码种，应用激活后有一段活跃时间，在这个活跃时间连接IM服务，接收下来消息，并弹出本地通知。

## 通知类型
一般情况下有2种推送，一种是本地通知，另外一种是远程推送。
1. 本地通知：指应用在后台处于激活状态，当有此用户的新消息时，消息会被收下来，然后本地弹出通知。
2. 远程推送：指应用处于冻结或者杀死状态，当有此用户的新消息时，消息无法被收下来，需要借助推送服务通知到用户。

本地通知和远程推送在手机上的表现很接近，都是应用放到后台，然后有人给此账号发送消息，通知栏弹出通知。实际上处理流程完全不同。本项目处理的是远程推送。***在处理通知问题时，首先要确认的是本地通知还是远程推送***。

## 接入推送
接入推送并不是简单得将推送服务跑起来即可，请详细阅读[接入推送流程](./push.md)

## 添加其它推送服务
由前面的介绍可以看出，推送子服务是独立于IM服务，而且客户端和服务器部分都是开源的，而且考虑到了扩展性，可以很容易地添加其它推送类型。具体步骤如下：
1. 必须理解推送的工作原理，知道流程是：客户端注册推送-》客户端注册推送成功得到deviceToken-》客户端调用设置deviceToken和类型，这两个数据被存储到IM服务。当IM服务需要推送时，IM服务打包推送信息（包括deviceToken和类型）请求到推送服务-》推送服务根据类型选择服务商推送数据。
2. 客户端扩展一个新的推送类型。
3. 客户端在应用启动时，添加处理这种推送类型的注册
4. 在注册成功后会得到deviceToken，调用IM SDK的setDeviceToken接口传人deviceToken和类型。
5. 推送服务添加对这种类型的处理。

## 使用到的开源代码
1. [TypeBuilder](https://github.com/ikidou/TypeBuilder) 一个用于生成泛型的简易Builder

## LICENSE
UNDER MIT LICENSE. 详情见LICENSE文件
