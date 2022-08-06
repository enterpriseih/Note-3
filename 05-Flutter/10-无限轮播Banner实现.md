# Flutter无限轮播Banner实现

[TOC]

![内容](图片\内容.png)

## 手势

​	Flutter中的手势系统有两个独立的层。第一层为原始指针(pointer)事件，它描述了屏幕上指针（例如，触摸、鼠标和触控笔）的位置和移动。 第二层为手势，描述由一个或多个指针移动组成的语义动作，如拖动、缩放、双击等。

### 原始指针事件

​	在移动端，各个平台或UI系统的原始指针事件模型基本都是一致，即：一次完整的事件分为三个阶段：手指按下、手指移动、和手指抬起，而高级的手势（如点击、双击、拖动等）都是基于这些原始事件的。

​	Flutter中可以使用Listener widget来监听原始触摸事件：

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      home: MainRoute(),
    );
  }
}

class MainRoute extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return _MainState();
  }
}

class _MainState extends State<MainRoute> {
  //定义一个状态，保存当前指针位置
  PointerEvent _event;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("主页"),
      ),
      body: Listener(
        child: Container(
          alignment: Alignment.center,
          width: 300.0,
          height: 150.0,
          child: Text(_event?.toString() ?? ""),
        ),
        onPointerDown: (PointerDownEvent event) =>
            setState(() => _event = event),
        onPointerMove: (PointerMoveEvent event) =>
            setState(() => _event = event),
        onPointerUp: (PointerUpEvent event) => setState(() => _event = event),
      ),
    );
  }
}
```

`PointerDownEvent`、`PointerMoveEvent`、`PointerUpEvent`都是`PointerEvent`的一个子类，`PointerEvent`类中包括当前指针的一些信息。

- `position`：它是鼠标相对于全局坐标的偏移。(左上角原点)。
- `delta`：两次指针移动事件（PointerMoveEvent）的距离。
- `pressure`：按压力度，如果手机屏幕支持压力传感器(如iPhone的3D Touch)，此属性会更有意义，如果手机不支持，则始终为1。
- `orientation`：指针移动方向，是一个角度值。



### 命中测试

​	当指针按下时，Flutter会对应用程序执行**命中测试(Hit Test)**，以确定指针与屏幕接触的位置存在哪些widget， 指针按下事件（以及该指针的后续事件）然后被分发到由命中测试发现的最内部的widget，然后从那里开始，事件会在widget树中向上冒泡，这些事件会从最内部的widget被分发到到widget根的路径上的所有Widget。

​	behavior属性决定子Widget如何响应命中测试，它的值类型为HitTestBehavior，这是一个枚举类，有三个枚举值：

- `deferToChild`：子widget会一个接一个的进行命中测试，如果子Widget中有测试通过的，则当前Widget通过。

> 指针事件作用于子Widget上时，父Widget也肯定可以收到该事件。

- `opaque`：不透明的。在命中测试时，将当前Widget当成不透明处理(即使本身是不可见、透明的)，最终的效果相当于当前Widget的整个区域都是点击区域。
- `translucent`：半透明的。当点击Widget时，widget可以接收到事件(无论是否可见)，子widget则需要点击到可见区域才能接收。

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      home: MainRoute(),
    );
  }
}

class MainRoute extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return _MainState();
  }
}

class _MainState extends State<MainRoute> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("主页"),
      ),
      body: Listener(
        behavior: HitTestBehavior.translucent,
        child: Container(
          alignment: Alignment.center,
          ///300x150不可见，只有Text可见
          ///默认情况下，点击Text区域才响应事件。点击空白区域无输出;点击Text才能响应
          ///设置为 opaque 后 则在300x300内都能响应，哪怕不可见。点击空白区域就能响应
          ///设置为 translucent 后 则在300x300能响应。也是点击空白区域就能响应
//            color: Colors.blue,///不设置颜色就是不可见
          width: 300.0,
          height: 300.0,
          child: Text(
            "点击",
          ),
        ),
        onPointerDown: (PointerDownEvent event) =>
            setState(() => debugPrint("响应")),
      ),
    );
  }
}
```

`opaque`和`translucent`的区别在于，后者是将透明区域视为半透明，这意味着能够完成"穿透"效果。

> 需要注意的是点击 **外部** 文字，因为文字本身不是透明，不会进行穿透效果。

```dart
 @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: Text("主页"),
        ),
        body: Stack(
          children: <Widget>[
            Listener(
              child: Container(
                width: 300.0,
                height: 300.0,
                color: Colors.blue,
                child: Center(child: Text("底部")),
              ),
              onPointerDown: (event) => print("down0"),
            ),
            Listener(
              child: Container(
                width: 100.0,
                height: 100.0,
                child: Center(child: Text("外部")),
              ),
              onPointerDown: (event) => print("down1"),
              behavior: HitTestBehavior.translucent, //穿透
            )
          ],
        ));
  }
```



### 手势识别

​	Android中存在事件冲突，Flutter其实也存在，但是官方的`GestureDetector`来解决这个问题。通常我们为了响应用户与设备屏幕交互就会使用这个手势Widget：`GestureDetector`。

> 包括之前使用的InkWell 内部实现也是GestureDetector

| 手势                                                         | 说明                                          |
| ------------------------------------------------------------ | --------------------------------------------- |
| `onTapDown`                                                  | 按下                                          |
| `onTapUp`                                                    | 抬起                                          |
| `onTapCancel`                                                | 触发了 onTapDown，但并没有完成一个 onTap 动作 |
| `onTap`                                                      | 点击动作                                      |
| `onDoubleTap`                                                | 双击                                          |
| `onLongPress`                                                | 长按                                          |
| `onScaleStart, onScaleUpdate, onScaleEnd`                    | 缩放                                          |
| `onVerticalDragDown, onVerticalDragStart, onVerticalDragUpdate, onVerticalDragEnd, onVerticalDragCancel` | 在竖直方向上移动                              |
| `onHorizontalDragDown, onHorizontalDragStart, onHorizontalDragUpdate, onHorizontalDragEnd, onHorizontalDragCancel` | 在水平方向上移动                              |
| `onPanDown, onPanStart, onPanUpdate, onPanEnd, onPanCancel`  | 拖曳                                          |

> 手势的识别比较复杂。分解动作：先点击再进行后续的手势操作(滑动、抬起)这时候点击下去会回调所有的`XXDown`方法。因为此时系统并不知道你需要进行的后续手势操作是什么。而如果是连贯的手势动作就只会回调对应的`Down`方法。
>
> 同时如果同时设置了拖拽手势参数与固定方向方法(水平、垂直)参数时候，那只会回调固定方向的方法。即固定方向优先级最高。

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text("主页"),
        ),
        body: _Drag(),
      ),
    );
  }
}

class _Drag extends StatefulWidget {
  @override
  _DragState createState() => new _DragState();
}

class _DragState extends State<_Drag> with SingleTickerProviderStateMixin {
  double _top = 0.0; //距顶部的偏移
  double _left = 0.0; //距左边的偏移

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: <Widget>[
        Positioned(
          top: _top,
          left: _left,
          child: GestureDetector(
            child: CircleAvatar(child: Text("A")),
            //手指滑动时会触发此回调
            onPanUpdate: (DragUpdateDetails e) {
              //用户手指滑动时，更新偏移，重新构建
              setState(() {
                _left += e.delta.dx;
                _top += e.delta.dy;
              });
            },
          ),
        )
      ],
    );
  }
}

```



### 手势冲突

​	如果我们同时监听水平和垂直方向的拖动事件，那么我们斜着拖动时哪个方向会生效？实际上取决于第一次移动时两个轴上的位移分量，哪个轴的大，哪个轴在本次滑动事件竞争中就胜出。例如，假设有一个ListView，它的第一个子Widget也是ListView，如果现在滑动这个子ListView，这时只有子Widget会动，因为这时子Widget会胜出而获得滑动事件的处理权。

​	识别水平和垂直方向的拖动手势，当用户按下手指时就会触发竞争（水平方向和垂直方向），一旦某个方向“获胜”，则直到当次拖动手势结束都会沿着该方向移动。

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text("主页"),
        ),
        body: Test(),
      ),
    );
  }
}

class Test extends StatefulWidget {
  @override
  TestState createState() => TestState();
}

class TestState extends State<Test> {
  double _top = 0.0;
  double _left = 0.0;

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: <Widget>[
        Positioned(
          top: _top,
          left: _left,
          child: GestureDetector(
            child: CircleAvatar(child: Text("A")),
            //垂直方向拖动事件
            onVerticalDragUpdate: (DragUpdateDetails details) {
              setState(() {
                _top += details.delta.dy;
              });
            },
            onHorizontalDragUpdate: (DragUpdateDetails details) {
              setState(() {
                _left += details.delta.dx;
              });
            },
          ),
        )
      ],
    );
  }
}

```

​	

## 自定义Widget

​	当Flutter提供的现有Widget无法满足我们的需求，或者我们为了共享代码需要封装一些通用Widget，这时我们就需要自定义Widget。自定义Widget主要有两种方式：自绘与组合封装。

### 自绘

​	对于一些复杂或不规则的UI，我们可能无法使用现有Widget组合的方式来实现。在Flutter中，提供了一个`CustomPaint`画笔，它可以结合一个画家`CustomPainter`来实现绘制自定义图形。

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new MaterialApp(
      title: 'Flutter Demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text("主页"),
        ),
        body: GradientCircularProgressRoute(),
      ),
    );
  }
}

class GradientCircularProgressRoute extends StatefulWidget {
  @override
  GradientCircularProgressRouteState createState() {
    return  GradientCircularProgressRouteState();
  }
}

class GradientCircularProgressRouteState
    extends State<GradientCircularProgressRoute>  {
  @override
  Widget build(BuildContext context) {
    //返回画笔
    return CustomPaint(
      painter: MyPainter(50.0),
    );
  }
}

class MyPainter extends CustomPainter {
  MyPainter(this.radius);

  double radius;

  @override
  void paint(Canvas canvas, Size size) {
    ///根据半径计算大小
    size = Size.fromRadius(radius);
    var paint = Paint() //创建一个画笔并配置其属性
      ..isAntiAlias = true //是否抗锯齿
      ..style = PaintingStyle.fill //画笔样式：填充
      ..color = Colors.blue //画笔颜色
      ..strokeWidth = 3.0; //画笔的宽度

    ///画一个实心圆
    Rect rect =
    Rect.fromCircle(center: size.center(Offset.zero), radius: radius);
    canvas.drawCircle(rect.center, radius, paint);
  }


  /// 返回true来重绘，反之则应返回false不需要重绘。
  @override
  bool shouldRepaint(MyPainter oldDelegate) {
    if(oldDelegate.radius != radius){
      return true;
    }
    return false;
  }
}
```



### 组合

​	这种方式是通过拼装其它低级别的Widget来组合成一个高级别的Widget，例如`Container`就是一个组合Widget，它是由`DecoratedBox、ConstrainedBox、Transform、Padding、Align`等组成。



## 自定义BannerView

> 代码



### Notification机制

内容引用自：

[Notification]: https://book.flutterchina.club/chapter8/notification.html

​	

Notification是Flutter中一个重要的机制，在Widget树中，每一个节点都可以分发通知，通知会沿着当前节点（context）向上传递，所有父节点都可以通过NotificationListener来监听通知，Flutter中称这种通知由子向父的传递为“通知冒泡”（Notification Bubbling），这个和用户触摸事件冒泡是相似的，但有一点不同：通知冒泡可以中止，但用户触摸事件不行。

Flutter中很多地方使用了通知，如可滚动(Scrollable) Widget中滑动时就会分发ScrollNotification，而Scrollbar正是通过监听ScrollNotification来确定滚动条位置的。除了ScrollNotification，Flutter中还有SizeChangedLayoutNotification、KeepAliveNotification 、LayoutChangedNotification等。下面是一个监听Scrollable Widget滚动通知的例子：

```dart
NotificationListener(
  onNotification: (notification){
    //print(notification);
    switch (notification.runtimeType){
      case ScrollStartNotification: print("开始滚动"); break;
      case ScrollUpdateNotification: print("正在滚动"); break;
      case ScrollEndNotification: print("滚动停止"); break;
      case OverscrollNotification: print("滚动到边界"); break;
    }
  },
  child: ListView.builder(
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(title: Text("$index"),);
      }
  ),
);
```

上例中的滚动通知如ScrollStartNotification、ScrollUpdateNotification等都是继承自ScrollNotification类，不同类型的通知子类会包含不同的信息，比如ScrollUpdateNotification有一个`scrollDelta`属性，它记录了移动的位移，其它通知属性读者可以自己查看SDK文档。

#### 自定义通知

除了Flutter内部通知，我们也可以自定义通知，下面我们看看如何实现自定义通知：

1. 定义一个通知类，要继承自Notification类；

   ```dart
   class MyNotification extends Notification {
     MyNotification(this.msg);
     final String msg;
   }
   ```

2. 分发通知。

   Notification有一个`dispatch(context)`方法，它是用于分发通知的，我们说过context实际上就是操作Element的一个接口，它与Element树上的节点是对应的，通知会从context对应的Element节点向上冒泡。

下面我们看一个完整的例子：

```dart
class NotificationRoute extends StatefulWidget {
  @override
  NotificationRouteState createState() {
    return new NotificationRouteState();
  }
}

class NotificationRouteState extends State<NotificationRoute> {
  String _msg="";
  @override
  Widget build(BuildContext context) {
    //监听通知  
    return NotificationListener<MyNotification>(
      onNotification: (notification) {
        setState(() {
          _msg+=notification.msg+"  ";
        });
      },
      child: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
//          RaisedButton(
//           onPressed: () => MyNotification("Hi").dispatch(context),
//           child: Text("Send Notification"),
//          ),  
            Builder(
              builder: (context) {
                return RaisedButton(
                  //按钮点击时分发通知  
                  onPressed: () => MyNotification("Hi").dispatch(context),
                  child: Text("Send Notification"),
                );
              },
            ),
            Text(_msg)
          ],
        ),
      ),
    );
  }
}

class MyNotification extends Notification {
  MyNotification(this.msg);
  final String msg;
}
```

上面代码中，我们每点一次按钮就会分发一个`MyNotification`类型的通知，我们在Widget根上监听通知，收到通知后我们将通知通过Text显示在屏幕上。

> 注意：代码中注释的部分是不能正常工作的，因为这个`context`是根Context，而NotificationListener是监听的子树，所以我们通过`Builder`来构建RaisedButton，来获得按钮位置的context。



## 练习作业

自己实现一个自定义Widget，形式不限，效果不限。如饼图、折线图等。

![练习作业](图片\练习作业.png)