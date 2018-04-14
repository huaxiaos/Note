# 名词

## ViewRootImpl

ViewRootImpl的核心任务是与WindowManagerService进行通信

一个Activity对应一个唯一的ViewRootImpl

ViewRootImpl中有以下几个重要的变量

- View mView 持有当前的View对象
- IWindowSession mWindowSession 用于WMS通信，这个变量通过WMS的openSession方法创建
- W mWindow 通过IWindowSession.add()方法提供，WMS可以通过这个对象与ViewRootImpl进行双向通信

## WindowManagerService

与ActivityManagerService对象类似

## WindowManagerImpl

```
public final class WindowManagerImpl implements WindowManager
```

WindowManagerImpl是WindowManager这个接口的实现类，Window这一层的功能之一，就是与WMS进行通信，由于一个应用中可能会有多个Window，每个Window都与WMS进行单独通信，会造成资源浪费和混乱，而这个WindowManagerImpl就是统一进行管理的类

WindowManagerImpl在每个应用进程中只有一个实例

## WindowManagerGlobal

4.3以前的API中，View[]、ViewRootImpl[]、WindowManager.LayoutParams[]等变量是保存在WindowManagerImpl中的，4.3以后，使用WindowManagerGlobal来统一管理这些变量

WindowManagerGlobal也是一个单例

## WindowManager

```
public interface WindowManager extends ViewManager
```

WindowManager是一个接口类，继承自ViewManager

## Window

```
public abstract class Window
```

Window是一个抽象类，根据不同的产品可以有不同的实现类，在Activity.attach方法中通过PolicyManager.makeNewWindow进行创建。目标版本，默认生成的都是PhoneWindow

## PhoneWindow

Window的子类

# Activity & Window & View

Activity并不直接参与View的管理，而是通过Window来管理View的，Activity相当于一个控制单元。