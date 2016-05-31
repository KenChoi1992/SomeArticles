本文旨在指导 React Native 初学者如何使用 React Native 初步构建 Android 应用。在看本文之前，你应该具备了理解 React Native 的基本概念，以及搭建好了所需的环境（如果没有，参考官方网站）。接下来，我们一起来实现一个简单的 PushDemoApp，借助 jpush-android-sdk 就可以实现推送功能。**PushDemoApp 的[源码](https://github.com/jpush/jpush-react-plugin)**。

我们先来看一下 PushDemoApp 最终的界面效果：

![PushDemo](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react-native-push-demo.gif)

怎么样，是不是很心动呢，接下来我们来看看如何一步步打造一个 React Native 应用吧。
首先创建一个 Project，在命令行中输入
>react-native init PushDemo

这样就创建了一个名为 PushDemo 的项目，使用 Android Studio 打开 PushDemo/android 项目，接下来改造 android 的工程结构以兼容 Eclipse，如图所示：

![工程结构图](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react1.png)

并且在 build.gradle 加上相关配置，此处不再赘述，可以参考 github 上的源码。

##配置入口
React Native 应用有 Native 和 JS 两个入口：MainActivity 以及 index.android.js。
####Native 入口
Android 的入口是在 MainActivity 中实例化 ReactInstanceManager，并通过 ReactRootView 启动 App。

```
mReactRootView = new ReactRootView(this);
 mReactInstanceManager = ReactInstanceManager.builder()
    .setApplication((Application) PushDemoApplication.getContext())
    .setBundleAssetName("index.android.bundle")
    .setJSMainModuleName("react-native-android/index.android")
    .addPackage(new MainReactPackage())
    .addPackage(new CustomReactPackage())
    .setUseDeveloperSupport(BuildConfig.DEBUG)
    .setInitialLifecycleState(LifecycleState.RESUMED)
    .build();
mReactRootView.startReactApplication(mReactInstanceManager, "PushDemoApp", null);
```
上面的 setBundleAssetName("index.android.bundle")，如果没有生成 index.android.bundle 文件，可以在 app 目录下新建一个 assets 文件夹，然后在命令行中启动 packager（使用 npm start 命令即可），启动后再输入以下命令生成 bundle 文件：
```
curl "http://localhost:8081/index.android.bundle?platform=android" -o "android/app/assets/index.android.bundle"
```
上面 -o 后面的路径即为存放 index.android.bundle 的路径，当然你也可以修改这个路径。

setJSMainModuleName("")这一句里面的参数则是 index.android.js 所在的文件路径，这个路径是相对于 package.json 的路径，在本例中则把 index.android.js 放在了 react-native-android 目录下。

####JS 入口
打开 index.android.js 文件（推荐使用 Sublime Text 开发 JS，有丰富的插件支持），可以看到最下面调用了 AppReistry 来注册 JS 入口：
```
React.AppRegistry.registerComponent("PushDemoApp", () => PushDemoApp);
```
要注意上面函数的第一个参数即为在 MainActivity 中 mReactRootView.startReactApplication()方法中的第二个参数。
##编写 JS
配置好入口后，接下来我们就可以开始编写 JS 了。我们从 index.android.js 文件开始讲解相关语法以及布局。首先看到第一句： ‘use strict’;放在第一行表明整个脚本都使用了 JS 的严格模式，只需要记住开发的时候一般都使用严格模式就行了。
```
import React from 'react-native';
import PushActivity from './push_activity';
import SetActivity from './set_activity';
import WebActivity from './web_activity';
```
import from 这是 ES6 的语法，关 于React Native 的代码规范，推荐[参考这个](https://github.com/airbnb/javascript/tree/master/react)
。本例所使用的都是 ES6 语法。from 后面是所需文件的相对路径，和 ES5 中的 require 类似，
可以[参考这个](http://www.ruanyifeng.com/blog/2015/05/require.html)。
```
var {
  BackAndroid,
  Component,
  Text,
  TextInput,
  View,
  Navigator,
} = React;
```
上面的代码声明了 React 的一些组件，下面就可以直接使用了。
```
  constructor(props) {
    super(props);
    
    this.state = {
       tag: '',
    }
    this.renderScene = this.renderScene.bind(this);
  }
```
这个构造函数可以用来做一些初始化的动作，使用 this.state = {}替代了 ES5 中 getInitialState 函数。React Native 较为出彩的一点就是将所有的组件都看成了状态机，通过设置组件的状态或者属性就可以触发 render 函数重新渲染，实现刷新界面的效果。
```
<Text style = { styles.btnText }
  { this.state.appKey }
</Text>
```
上面 Text 相当于 Android中 的 TextView 控件，Text 控件包裹的内容引用了一个状态：appKey，appKey 在构造函数中被初始化为 abc，那么这个 Text 就会显示 abc 这个字符串。使用 this.setState 方法就可以改变状态，比如：
```
  this.setState({ appKey: '123' });
```
这样 Text 就会刷新，重新显示为123。还可以将状态作为属性传递给其他组件，比如：
```
<Actionbar style = { styles.actionbar }>
    onselect = { this.onSelectMenu }
    currentPage = { this.state.page }
</Actionbar>
```
上面的代码将 this.onSelectMenu 以及 this.state.page 作为属性传递到 Actionbar 这个组件了，前者是一个函数，后者是一个状态，这样在 Actionbar 中就可以通过 this.props.onselect 以及 this.props.currentPage 来获得这两个属性。当这两个属性变化时，也会触发 render 函数重新渲染。在 Redux 架构中，通过改变属性来刷新界面非常常见，后面会讲到这个架构。

接下来是这一句
>this.renderScene = this.renderScene.bind(this);

在 ES5 下，React.createClass 会把所有的方法都 bind 一遍，这样可以提交到任意的地方作为回调函数，而 this 不会变化。但官方现在逐步认为这反而是不标准、不易理解的。在 ES6 下，你需要通过 bind 来绑定 this 引用，或者使用箭头函数（它会绑定当前 scope 的 this 引用）
来调用。关于ES5与ES6的写法对照可以[参考这个](http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8/2)

接下来先看到 componentDidMount() 和 componentWillUnmount() 这两个函数，这两个是组件的生命周期函数，在组件进入生命周期的不同阶段时
自动调用。关于组件生命周期的介绍可以[参考这篇博客](http://blog.csdn.net/sbsujjbcy/article/details/49925781)。在
 componentDidMount() 声明了点击 Android 的返回键时从 navigator 的栈中弹出一个页面，类似于 Android 中的 onBackPress() 方法。

接下来是 render() 函数，这个函数就是绘制界面的地方。React Native 可以用 Html 或 JSX 来编写界面，推荐使用官方推荐的 JSX 语法。
```
render() {
      return (
          <Navigator
              initialRoute = { {name: 'pushActivity' }}
              configureScene = { this.configureScene }
              renderScene = { this.renderScene } />
        );
    }
```
可以看到这里仅仅是返回了一个 Navigator。
####Navigator
React Native 使用 Navigator 来控制页面的跳转。Navigator 有3个属性：

- initialRoute 声明了初始要显示的界面，这里是传了一个 route name 过去，即 “pushActivity”；

- configureScene 指定了页面跳转的动画，查看 Navigator.SceneConfigs 来获取默认的动画和更多的场景配置选项；

- renderScene 声明了页面该如何跳转。
```
renderScene(router, navigator) {
    var Component = null;
    this.navigator = navigator;
    switch(router.name) {
      case "pushActivity":
        Component = PushActivity;
        break;
      case "setActivity":
        Component = SetActivity;
        break;
      case "webActivity":
        Component = WebActivity;
    }
    //将navigator作为属性传给其他页面，这样在其他页面中就可以使用this.props.navigator拿到navigator了
    return <Component navigator = { navigator } />
  }
```
renderScene() 有两个参数，router 对象指定要跳转的界面，router.name 可以拿到刚才我们在 initialRoute 时传过来的 name，此处仅仅将 name 传过来，然后根据 name 来返回 Component，后面会看到其他写法。navigator 可以作为属性传给其他页面，还可以带一些参数传到其他页面，相当于 Android 中的 Intent。比如：
```
return <Component
          navigator = { navigator }
          params = {
            name: '',
            title: '',
          }
        />
```
这样在其他页面可以通过 this.props.name 以及 this.props.title 来拿到这两个值，相当于 getIntent()。
关于页面的启动模式，即相当于 Android 中 Activity 的4种启动模式以及 flag 的设置，在 React Native 中则是在跳转时调用 Navigator 的内部方法来控制，如下：

- getCurrentRoutes() - returns the current list of routes, 返回当前栈中的所有路由
- jumpBack() - Jump backward without unmounting the current scene，在保留当前场景的情况下返回到上一个场景
- jumpForward() - Jump forward to the next scene in the route stack 回到刚才 jumpBack() 之前的场景
- jumpTo(route) - Transition to an existing scene without unmounting 跳转到当前栈中的某个场景并且不会卸载掉之前的场景
- push(route) - Navigate forward to a new scene, squashing any scenes that you could jumpForward to 往栈中 push 一个新场景
- pop() - Transition back and unmount the current scene 在栈中卸载掉当前场景，并返回上一个场景
- replace(route) - Replace the current scene with a new route 用一个新路由替换掉当前场景
- replaceAtIndex(route, index) - Replace a scene as specified by an index 替换指定索引的路由场景
- replacePrevious(route) - Replace the previous scene 替换掉上一个场景
- resetTo(route) - Navigate to a new scene and reset route stack 跳转到一个新场景，并且重置路由栈，相当于 Android中的 new Task
- immediatelyResetRouteStack(routeStack) - Reset every scene with an array of routes 用一个新路由栈替换之前的路由栈
- popToRoute(route) - Pop to a particular scene, as specified by its route. All scenes after it will be unmounted 跳转到指定的场景，在这个场景之上的都会被卸载掉。相当于 Android 中的 CLEAR_TOP 标志
- popToTop() - Pop to the first scene in the stack, unmounting every other scene 跳转到第一个场景，其他的场景将会被卸载

####布局

我们再来看一下 push_activity.js 这个类，这是应用的启动界面，先来看一下 render() 方法，render() 中 return 的内容相当于 Android 中的 xml 布局，React Native 使用了 css-layout，实现了 flexbox，Flexbox 布局默认是线性布局即 Android 中的 LinearLayout，PushActivity 最外面由 ScrollView 包裹，style 属性指定了该控件使用了哪种 style，如果一个控件没有声明高度或宽度，则默认填充满父布局。使用 StyleSheet.create 来定义样式，可以看到样式定义所用的语法是 JSON 格式的：
```
var styles = React.StyleSheet.create({
  container: {  
    flex: 1,
  },
});

```
flex:1 这个也相当于 match_parent。**需要注意的是，在 React Native 的样式中中，如果没有声明，数值的默认单位都是 dp，而不是 px**，下面讲解布局相关的属性。

flexDirection:指定排列方向，有 row, column 这两个值，分别是水平及垂直排列，默认为垂直排列。

*alignItems*: 可能的取值: 

- flex-start  控件在x轴的起点对齐；但在水平布局中为在y轴的起点对齐
- center 控件在x轴的中点对齐；在水平布局中为在y轴的中点对齐
- flex-end 控件在x轴的终点对齐；但在水平布局中为在y轴的终点对齐
- stretch 默认值，占满容器的高度；如果为水平方向则占满宽度。

*justifyContent*: 可能的取值: 

- flex-start 在垂直布局中为上对齐（默认值）；在水平布局中为左对齐（默认值）
- center 居中
- flex-end 在垂直布局中为下对齐；在水平布局中为右对齐
- space-between 在垂直布局中以高为基准，控件两端对齐，控件之间的间隔相等；在水平布局中以宽为基准，控件两端对齐，控件之间间隔相等
- space-around 在垂直布局中以高为基准，控件之间的间隔相等，控件与边界的间隔为控件之间间隔的一半；在水平布局中以宽为基准，控件之间的间隔相等，控件与边界的间隔为控件之间间隔的一半

![如下图所示](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/ReactNativeFlexBox.png)

*alignSelf* 其效果与 alignItems 一样，但针对个别控件。如果某个控件声明了 alignSelf 属性，则会覆盖父容器的 alignItems 属性。

*position* :可能的取值：
- relative 相对位置（默认值）
- absolute 绝对位置

这两个属性一般会搭配 left，top，right，bottom 这4个属性来使用，分别表示控件的左上右下边缘距离y轴或x轴的距离。

还有一些属性是与控件相关的：

**背景颜色**

1. backgroundColor //所有涉及颜色的都是字符串类型的 rgb 格式，如#ffffff

**边框**

1. borderBottomWidth //底部边框宽度
2. borderLeftWidth  //左边边框宽度
3. borderRightWidth //右边边框宽度
4. borderTopWidth //顶部边框宽度
5. borderWidth  //所有边框宽度
6. borderTopLeftRadius //左上圆角
7. borderTopRightRadius //右上圆角
8. borderBottomLeftRadius //左下圆角
9. borderBottomRightRadius //右下圆角
10. borderRadius //圆角
11. borderBottomColor //底边框颜色
12. borderLeftColor //左边框颜色
13. borderRightColor //右边框颜色
14. borderTopColor //上边框颜色
15. borderColor //边框颜色

**外边距**

1. marginBottom
2. marginLeft
3. marginRight
4. marginTop
5. marginVertical
6. marginHorizontal
7. margin

**内边距**

paddingBottom  
paddingLeft  
paddingRight  
paddingTop  
paddingVertical
paddingHorizontal  
padding

**宽高**

1. width
2. height

**字体相关**

1. color 字体颜色
2. fontFamily 字体族
3. fontSize 字体大小
4. fontStyle 字体样式，正常，倾斜等，值为enum('normal', 'italic')
5. fontWeight 字体粗细，值为 enum("normal", 'bold', '100', '200', '300', '400', '500', '600', '700', '800', '900')
6. letterSpacing 字符间隔
7. lineHeight 行高
8. textAlign 字体对齐方式，值为 enum("auto", 'left', 'right', 'center', 'justify')
9. textDecorationLine 字体修饰，上划线，下划线，删除线，无修饰，值为 enum("none", 'underline', 'line-through', 'underline underline-through')
10. textDecorationStyle enum("solid", 'double', 'dotted', 'dashed') 修饰的线的类型
11. textDecorationColor 修饰的线的颜色
12. writingDirection enum("auto", 'ltr', 'rtl') 字体显示方向
 
**图片相关**

1. resizeMode enum('cover', 'contain', 'stretch') conver 是指按照图片实际大小显示，超出部分裁剪，默认值；contain 是无论如何都将图片显示在控件中，如果超出则等比例缩小并居中显示；stretch 是指将图片进行拉伸，填充满控件
2. overflow enum('visible', 'hidden') 超出部分是否显示，hidden 为隐藏
3. tintColor 着色，rgb字符串类型
4. opacity 透明度

在 render() 中，使用 JSX 的写法，用一对\<View\>\</View\>标签来包裹一个控件，这相当于 Html 中的\<div\>\</div\>标签。下面给出一些 React Native 与 Android 控件对照：

|**React Native**|**Android**|
|:---:|:---:|
|Text|TextView|
|TouchableHighlight|Button|
|TextInput|EditText|
|image|ImageView|
|ScrollView|ScrollView|
|ListView|ListView|

React Native 还实现了一些比较常用的控件，如 Switch、Picker、ToolbarAndroid、ViewPagerAndroid 等。这些控件就不再一一讲解了，有很多文章可以参考，官网也有[demo](https://github.com/facebook/react-native/tree/master/Examples/UIExplorer)

##JS 调用 Native
下面来讲解一下 JS 如何调用 Native。如果我们的应用是混合的 React Native 应用(使用了第三方 jar，如本例中的 jpush-sdk)，那么在 JS 中调用 sdk 中的接口是非常常见而且也是必要的。使用 NativeModule 就可以达到这种目的。

要声明一个NativeModule，需要以下几个步骤。

- **定义一个 NativeModule 类，继承自 ReactContextBaseJavaModule。**

```
public class PushHelperModule extends ReactContextBaseJavaModule {

    private static String TAG = "PushHelperModule";
    private Context mContext;

    public PushHelperModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    @Override
    public boolean canOverrideExistingModule() {
        return true;
    }
    
    @Override
    public String getName() {
        return "PushHelper";
    }
}

```
声明一个 NativeModule 类时，需要重写 canOverrideExistingModule() 及 getName() 两个方法，在 getName 中返回的字符串即为在 JS 中引用的类，即在JS中使用 PushHelper 来引用 PushHelperModule 这个类。现在可以在 PushHelperModule 中添加 ReactMethod 了：
```
@ReactMethod
    public void init(Callback successCallback, Callback errorCallback) {
        try {
            JPushInterface.init(PushDemoApplication.getContext());
            successCallback.invoke("init Success!");
            Log.i("PushSDK", "init Success !");
        } catch (Exception e) {
            errorCallback.invoke(e.getMessage());
        }

    }
```
使用 @ReactMethod 标签来表明这是一个 React 方法，这样就可以在 JS 中直接调用，而且这个方法必须为 public，返回类型为 void。上面的方法还是用了 Callback（com.facebook.react.bridge.Callback），CallBack 可以用来在 Native 中回调 JS 中的方法。结合本例，init 方法中调用了 JPushInterface.init(context) 这个接口，接着执行了 callback.invoike()，这个方法就可以回调到 JS 了。invoke() 可以带参数，String 或者 Map 都行。可以有多个 Callback，相应地在 JS 中要带多个参数。

>push_activity.js

```
  onInitPress() {
    PushHelper.init( (success) => {
      ToastAndroid.show(success, ToastAndroid.SHORT);
    }, (error) => {
      ToastAndroid.show(error, ToastAndroid.SHORT);
    });
  }

```
在 JS 中使用 () => {} 箭头函数就可以接收回调函数了。

- **新建一个类实现 ReactPackger，然后在将刚才定义的 NativeModule 加进来：**

```
public class CustomReactPackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> result = new ArrayList<>();
        result.add(new PushHelperModule(reactContext));
        result.add(new ToastModule(reactContext));
        result.add(new JSHelperModule(reactContext));
        return result;
    }

    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
        return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        List<ViewManager> result = new ArrayList<>();
        result.add(new ReactTextManager());
        return result;
    }
}

```
在重写的 createNativeModules() 方法中将 PushHeperModule 加到 List，然后返回这个 List 就行了。其他的模块如果为空可以返回 Collections.emptyList()。

- **在 MainActivity 中将 packger 加到 ReactInstanceManager 的构造方法中**

```
mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication((Application) PushDemoApplication.getContext())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModuleName("react-native-android/index.android")
                .addPackage(new MainReactPackage())
                .addPackage(new CustomReactPackage())
                .setUseDeveloperSupport(BuildConfig.DEBUG)
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        mReactRootView.startReactApplication(mReactInstanceManager, "PushDemoApp", null);

```

- **在 JS 中用 NativeModules 引用**

```
var {
  NativeModules,
} = React;
var PushHelper = NativeModules.PushHelper;
```
之后就可以使用 PushHelper 直接调用使用了 @ReactMethod 标签的方法了。需要注意的是，如果在 Native 添加了新的类，必须重新编译运行一次 App，只是 Reload JS 是不能刷新的。

##Native 调用 JS
在 Native 中调用 JS，最简单的方法是使用 RCTDeviceEventEmitter。
打开 MainActivity，在 MessageReceiver 的o nReceive() 中可以看到这段代码：
```
Assertions.assertNotNull(mReactInstanceManager.getCurrentReactContext())        
  .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)        
  .emit("receivePushMsg", messge);
```
通过 getCurrentReactContext.getJSModule() 方法找到 RCTDeviceEventEmitter 这个模块，然后调用 emit 方法调用 JS 中的方法，emit() 方法的第一个参数即为在 JS 中定义的接收事件名，第二个参数可以传递一个对象到 JS 中。接下来在 JS 中接收这个事件：
>push_activity.js

```
componentWillMount() {
  DeviceEventEmitter.addListener('receivePushMsg', (data) => {
        this.setState({ pushMsg: data });
      });
}
```
在组件注册的生命周期函数中使用 DeviceEventEmitter.addListener() 来接收 RCTDeviceEventEmitter 的 emit 事件即可。然后在组件卸载的生命周期函数中移除 Listener：
```
componentWillUnmount() {
      DeviceEventEmitter.removeAllListeners();
}
```

##运行
下面来说一下如何运行。

在 init 完 Project 后，就可以直接在命令行中使用

>react-native run-android

命令运行了。这时会自动弹出一个窗口来执行 React Packger。

![React Packger](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react2.png)

如果使用模拟器，推荐使用 Genymotion，只要注册一个账号就可以了。运行后如果出现

![错误](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react3.png)

不要方，点击一下 Reload JS 即可，如果出现了 Unable to download JS bundle 错误：

![错误2](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react4.png)

首先检查一下设备和电脑的网络，确保两者在同一个 Wifi 环境下。在模拟器上点击下面的三角按钮，在弹出的菜单中点击 Menu 按钮，如图：

![menu](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react5.png)

就可以唤出设置对话框：

![dialog](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react6.png)

在真机中，只需要晃动手机，即可弹出上面的对话框。然后点击 Dev Settings 选项进入设置界面。

![setting](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react7.png)

然后点击 Debug server host & port for device，在弹出的对话框中，输入电脑连接的 IP 地址加上8081端口即可，如：

> 192.168.1.1:8081

然后点击 OK，返回，重新唤出设置对话框，点击 Reload JS 选项，这时应该可以正确运行了。

有问题可与我联系 QQ: 992649726, 或者发邮件。后续会发表 ListView 的用法以及在 React Native 中如何使用 Redux 架构。



