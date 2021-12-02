# Flutter

1. Widget - StatefulWidget - StatelessWidget、StatefulWidget 生命周期


initState()：Widget 初始化当前 State，在当前方法中是不能获取到 Context 的，如想获取，可以试试 Future.delayed()
didChangeDependencies()：在 initState() 后调用，State对象依赖关系发生变化的时候也会调用。
deactivate()：当 State 被暂时从视图树中移除时会调用这个方法，页面切换时也会调用该方法，和Android里的 onPause 差不多。
dispose()：Widget 销毁时调用。
didUpdateWidget：Widget 状态发生变化的时候调用。


2. Widget、Element、RenderObject 渲染树

Widget：仅用于存储渲染所需要的信息。

RenderObject：负责管理布局、绘制等操作。

Element：才是这颗巨大的控件树上的实体。


3. mixin 关键字


4. 与原生工程的配合方案 - 比如 Alibaba Flutter Boost


5. Flutter 是如何与原生Android、iOS进行通信的？
Flutter 通过 PlatformChannel 与原生进行交互，其中 PlatformChannel 分为三种：
BasicMessageChannel：用于传递字符串和半结构化的信息。
MethodChannel：用于传递方法调用（method invocation）。
EventChannel: 用于数据流（event streams）的通信。
同时 Platform Channel 并非是线程安全的


6. 图像渲染引擎 Skia






# Android 多渠道、多环境打包方案

# Android 异常捕获方案

# 设计模式

比如

1. 责任链
2. 代理
3. 观察者

# HTTPS

防止 APP 被代理软件抓包

# 跨域请求解决方案