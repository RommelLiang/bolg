
### Platform Channel简介
Flutter引入Platform Channel机制来支持不同平台的API调用。在Flutter中,提供了三种Platform Channel用来支持和平台之间数据的传递:

* BasicMessageChannel：支持字符串和半结构化的数据传递，可以通过BasicMessageChannel来获取Native项目的图标等资源
* MethodChannel：支持传递方法调用,Flutter主动调用Native的方法，并获取相应的返回值。既可以从Flutter发平台发起方法调用,也可以从平台代码向Flutter发起调用
* EventChannel：支持数据流通信，传递事件。收到消息后无法回复此次消息，通常用于Native向Dart的通信
### 使用方法
#### BasicMessageChannel
Android端：
```
BasicMessageChannel mBasicMessageChannel = new BasicMessageChannel(getFlutterView(), "basic_channel", StringCodec.INSTANCE);
mBasicMessageChannel.setMessageHandler(new BasicMessageChannel.MessageHandler() {
    //接受消息
    @Override
    public void onMessage(Object o, BasicMessageChannel.Reply reply) {
        Log.e("basic_channel", "接收到来自flutter的消息:"+o.toString());
        reply.reply("回馈消息");
    }
});
//发送消息
mBasicMessageChannel.send("向flutter发送消息");
//发送消息并接受flutter的回馈
mBasicMessageChannel.send("向flutter发送消息", new BasicMessageChannel.Reply() {
            @Override
            public void reply(Object o) {
                
            }
});
```
Flutter端：
```
const basicMessageChannel = const BasicMessageChannel('basic_channel', StringCodec());
//接受并回复消息
basicMessageChannel.setMessageHandler(
      (String message) => Future<String>(() {
            setState(() {
              this.message = message;
            });
            return "回复native消息";
      }),
);
//发送消息
basicMessageChannel.send("来自flutter的message");
//flutter并没有发送并接受回复消息的`send(T message, BasicMessageChannel.Reply<T> callback)`方法
```
#### MethodChannel
Android端：
```
MethodChannel mMethodChannel = new MethodChannel(getFlutterView(), "method_channel");
mMethodChannel.setMethodCallHandler(new MethodChannel.MethodCallHandler() {
    //响应flutter端的调用
    @Override
    public void onMethodCall(MethodCall methodCall, MethodChannel.Result result) {
        if (methodCall.method.equals("noticeNative")) {
            todo()
            result.success("接受成功");
        }
    }
});
//原生调用flutter
mMethodChannel.invokeMethod("noticeFlutter", "argument", new MethodChannel.Result() {
            @Override
            public void success(Object o) {
                //回调成功
            }
            @Override
            public void error(String s,String s1, Object o) {
                //回调失败
            }
            @Override
            public void notImplemented() {

            }
});
```
Flutter端：
```
const methodChannel = const MethodChannel('method_channel');
Future<Null> getMessageFromNative() async {
    //flutter调原生方法
    try {
      //回调成功
      final String result = await methodChannel.invokeMethod('noticeNative');
      setState(() {
        method = result;
      });
    } on PlatformException catch (e) {
      //回调失败
    }
  }
methodChannel.setMethodCallHandler(
      (MethodCall methodCall) => Future<String>(() {
            //响应原生的调用
          if(methodCall.method == "noticeFlutter"){
            setState(() {
              
            });
          }
      }),
); 
```
#### EventChannel
Android端：
```
EventChannel eventChannel = new EventChannel(getFlutterView(),"event_channel");
eventChannel.setStreamHandler(new EventChannel.StreamHandler() {
    @Override
    public void onListen(Object o, EventChannel.EventSink eventSink) {
        eventSink.success("成功");
        //eventSink.error("失败","失败","失败");
    }
    @Override
    public void onCancel(Object o) {
        //取消监听时调用
    }
});
```
Flutter端：
```
const eventChannel = const EventChannel('event_channel');
eventChannel.receiveBroadcastStream().listen(_onEvent,onError:_onError);
void _onEvent(Object event) {
    //返回的内容
}
void _onError(Object error) {
    //返回的回调
}
```

其中：Object args是传递的参数，EventChannel.EventSink eventSink是Native回调Dart时的会回调函数，eventSink提供success、error与endOfStream三个回调方法分别对应事件的不同状态
### 源码初探
#### Platform Channel基本结构
首先了解一下这三种Channel的代码：
##### BasicMessageChannel
```
class BasicMessageChannel<T> {
  const BasicMessageChannel(this.name, this.codec);
  final String name;
  final MessageCodec<T> codec;
  Future<T> send(T message) async {
    return codec.decodeMessage(await BinaryMessages.send(name, codec.encodeMessage(message)));
  }
  void setMessageHandler(Future<T> handler(T message)) {
    if (handler == null) {
      BinaryMessages.setMessageHandler(name, null);
    } else {
      BinaryMessages.setMessageHandler(name, (ByteData message) async {
        return codec.encodeMessage(await handler(codec.decodeMessage(message)));
      });
    }
  }
  void setMockMessageHandler(Future<T> handler(T message)) {
    if (handler == null) {
      BinaryMessages.setMockMessageHandler(name, null);
    } else {
      BinaryMessages.setMockMessageHandler(name, (ByteData message) async {
        return codec.encodeMessage(await handler(codec.decodeMessage(message)));
      });
    }
  }
}
```
##### MethodChannel
```
class MethodChannel {
  const MethodChannel(this.name, [this.codec = const StandardMethodCodec()]);
  final String name;
  final MethodCodec codec;
  void setMethodCallHandler(Future<dynamic> handler(MethodCall call)) {
    BinaryMessages.setMessageHandler(
      name,
      handler == null ? null : (ByteData message) => _handleAsMethodCall(message, handler),
    );
  }
  void setMockMethodCallHandler(Future<dynamic> handler(MethodCall call)) {
    BinaryMessages.setMockMessageHandler(
      name,
      handler == null ? null : (ByteData message) => _handleAsMethodCall(message, handler),
    );
  }
  Future<ByteData> _handleAsMethodCall(ByteData message, Future<dynamic> handler(MethodCall call)) async {
    final MethodCall call = codec.decodeMethodCall(message);
    try {
      return codec.encodeSuccessEnvelope(await handler(call));
    } on PlatformException catch (e) {
      returun ...
    } on MissingPluginException {
      return null;
    } catch (e) {
      return ...
    }
  }
  Future<T> invokeMethod<T>(String method, [dynamic arguments]) async {
    assert(method != null);
    final ByteData result = await BinaryMessages.send(
      name,
      codec.encodeMethodCall(MethodCall(method, arguments)),
    );
    if (result == null) {
      throw MissingPluginException('No implementation found for method $method on channel $name');
    }
    final T typedResult = codec.decodeEnvelope(result);
    return typedResult;
  }
}
```
##### EventChannel
```
class EventChannel {
  const EventChannel(this.name, [this.codec = const StandardMethodCodec()]);
  final String name;
  final MethodCodec codec;
  Stream<dynamic> receiveBroadcastStream([dynamic arguments]) {
    final MethodChannel methodChannel = MethodChannel(name, codec);
    StreamController<dynamic> controller;
    controller = StreamController<dynamic>.broadcast(onListen: () async {
      BinaryMessages.setMessageHandler(name, (ByteData reply) async {
        ...
      });
      try {
        await methodChannel.invokeMethod<void>('listen', arguments);
      } catch (exception, stack) {
        ...
      }
    }, onCancel: () async {
      BinaryMessages.setMessageHandler(name, null);
      try {
        await methodChannel.invokeMethod<void>('cancel', arguments);
      } catch (exception, stack) {
        ...
      }
    });
    return controller.stream;
  }
}
```
这三种Channel都有两个成员变量：

* name:表示Channel名字,用于区分不同Platform Channel的唯一标志，每个Channel使用唯一的name作为其唯一标志
* codec: 表示消息的编解码器,Flutter采用了二进制字节流作为数据传输协议:发送方需要把数据编码成二进制数据,接受方再把数据解码成原始数据.而负责编解码操作的就是Codec。
每个Channel中都使用到了`BinaryMessages`，它起到了信使的作用，负责将信息进行跨平台的搬运，是消息发送和接受的工具。

##### setMessageHandler
在创建好`BasicMessageChannel`后，让其接受来自另一平台的消息，`BinaryMessenger`调用它的`setMessageHandler`方法为其设置一个消息处理器,配合`BinaryMessenger`完成消息的处理以及回复；
##### send
在创建好`BasicMessageChannel`后，可以调用它的send方法向另一个平台传递数据。
##### setMethodCallHandler
设置用于在此`MethodChannel`上接收方法调用的回调
##### receiveBroadcastStream
设置广播流以接收此`EventChannel`上的事件
#### Handler
Flutter使用Handler处理Codec解码后的消息。三种Platform Channel相对应,Flutter中也定义了三种Handler:

* MessageHandler: 用于处理字符串或者半结构化消息,定义在BasicMessageChannel中.
* MethodCallHandler: 用于处理方法调用,定义在MethodChannel中.
* StreamHandler: 用于事件流通信,定义在EventChannel中

使用Platform Channel时，需要为其注册一个对应BinaryMessageHandler为其设置对应的Handler。二进制数据会被BinaryMessageHanler进行处理,首先使用Codec进行解码操作,然后再分发给具体Handler进行处理。

### 结语
欲更进一步了解Platform Channel设计与实现，可前往[深入Flutter技术内幕:Platform Channel设计与实现](http://lionoggo.com/2019/02/09/%E6%B7%B1%E5%85%A5Flutter%E6%8A%80%E6%9C%AF%E5%86%85%E5%B9%95_Platform%20Channel%E5%8E%9F%E7%90%86/?from=timeline&isappinstalled=0#Handler)，关注大神博客