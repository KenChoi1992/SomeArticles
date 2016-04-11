本文旨在指导React Native初学者如何使用React Native初步构建Android应用。在看本文之前，你应该具备了理解React Native的基本概念，
以及搭建好了所需的环境（如果没有，参考官方网站）。接下来，我们一起来实现一个简单的PushDemoApp，借助jpush-android-sdk就可以实
现推送功能。PushDemoApp的[源码](https://github.com/jpush/jpush-react-plugin)。我们先来看一下PushDemoApp最终的界面效果：

![PushDemo](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react-native-push-demo.gif)

怎么样，是不是很心动呢，接下来我们来看看如何一步步打造一个React Native应用吧。
首先创建一个Project，在命令行中输入
>react-native init PushDemo

这样就创建了一个名为PushDemo的项目，使用Android Studio打开PushDemo/android项目，接下来改造android的工程结构以兼容Eclipse，如图所示：

![工程结构图](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/react1.png)

并且在build.gradle加上相关配置，此处不再赘述，可以参考github上的源码。

React Native应用有Native和JS两个入口：MainActivity以及index.android.js。Android的入口是在MainActivity中实例化
ReactInstanceManager，并通过ReactRootView启动App。

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
上面的setBundleAssetName("index.android.bundle")，如果没有生成index.android.bundle文件，可以在app目录下新建一个assets目录，
然后在命令行中启动packager（使用npm start命令即可），启动后再输入以下命令生成bundle文件：
```
curl "http://localhost:8081/index.android.bundle?platform=android" -o "android/app/src/main/assets/index.android.bundle"
```
setJSMainModuleName("")这一句里面的参数则是index.android.js所在的文件路径，这个路径是相对于package.json的路径，在本例中则
把index.android.js放在了react-native-android目录下。

注意mReactRootView.startReactApplication()这个方法中的第二个参数，这个参数在JS的入口处注册App的时候调用到，即为
index.android.js如下方法中的第一个参数：
```
React.AppRegistry.registerComponent("PushDemoApp", () => PushDemoApp);
```
接下来我们打开index.android.js文件（推荐使用Sublime Text开发JS，有丰富的插件支持）。
首先看到第一句： ‘use strict’;放在第一行表明整个脚本都使用了JS的严格模式，只需要记住开发的时候一般都使用严格模式就行了。
```
import React from 'react-native';
import PushActivity from './push_activity';
import SetActivity from './set_activity';
import WebActivity from './web_activity';
```
import from这是ES6的语法，关于React Native的代码规范，推荐[参考这个](https://github.com/airbnb/javascript/tree/master/react)
。本例所使用的都是ES6语法。from后面是所需文件的相对路径，和ES5中的require类似，
可以[参考这个](http://www.ruanyifeng.com/blog/2015/05/require.html)
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
上面的代码声明了React的一些组件，下面就可以直接使用了。
```
  constructor(props) {
    super(props);
    this.renderScene = this.renderScene.bind(this);
  }
```
这个构造函数可以用来做一些初始化的动作，使用this.state = {}替代了ES5中getInitialState函数。React Native较为出彩的一点就是
将所有的组件都看成了状态机，通过设置组件的状态，触发render函数重新渲染，就可以实现刷新界面的效果。使用this.setState方法就
可以改变状态，后面会讲到这一点。
>this.renderScene = this.renderScene.bind(this);

在ES5下，React.createClass会把所有的方法都bind一遍，这样可以提交到任意的地方作为回调函数，而this不会变化。但官方现在逐步
认为这反而是不标准、不易理解的。在ES6下，你需要通过bind来绑定this引用，或者使用箭头函数（它会绑定当前scope的this引用）
来调用。关于ES5与ES6的写法对照可以[参考这个](http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8/2)

接下来先看到componentDidMount()和componentWillUnmount()这两个函数，这两个是组件的生命周期函数，在组件进入生命周期的不同阶段时
自动调用。关于组件生命周期的介绍可以[参考这篇博客](http://blog.csdn.net/sbsujjbcy/article/details/49925781)。在
componentDidMount()声明了点击Android的返回键时从navigator的栈中弹出一个页面，类似于Android中的onBackPress()方法。

接下来是render()函数，这个函数就是绘制界面的地方。React Native可以用Html或JSX来编写界面，推荐使用官方推荐的JSX语法。
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
这里仅仅是返回了一个Navigator。React Native使用Navigator来控制页面的跳转。Navigator有3个属性：
- initialRoute 声明了初始要显示的界面，这里是传了一个route name过去即“pushActivity”

- configureScene 指定了页面跳转的动画，查看Navigator.SceneConfigs来获取默认的动画和更多的场景配置选项。

- renderScene 声明了页面该如何跳转
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
renderScene()有两个参数，router对象指定要跳转的界面，router.name可以拿到刚才我们在initialRoute时传过来的name，此处仅仅将
name传过来，然后根据name来返回Component，后面会看到其他写法。navigator可以作为属性传给其他页面，还可以带一些参数传到其他页面，相当于Android中的Intent。比如：
```
return <Component
          navigator = { navigator }
          params = {
            name: '',
            title: '',
          }
        />
```
这样在其他页面可以通过this.props.name以及this.props.title来拿到这两个值，相当于getIntent()。
关于页面的启动模式，即相当于Android中Activity的4种启动模式以及flag的设置，在React Native中则是在跳转时调用Navigator的内部
方法来控制，如下：

- getCurrentRoutes() - returns the current list of routes
- jumpBack() - Jump backward without unmounting the current scene
- jumpForward() - Jump forward to the next scene in the route stack
- jumpTo(route) - Transition to an existing scene without unmounting
- push(route) - Navigate forward to a new scene, squashing any scenes that you could jumpForward to
- pop() - Transition back and unmount the current scene
- replace(route) - Replace the current scene with a new route
- replaceAtIndex(route, index) - Replace a scene as specified by an index
- replacePrevious(route) - Replace the previous scene
- resetTo(route) - Navigate to a new scene and reset route stack
- immediatelyResetRouteStack(routeStack) - Reset every scene with an array of routes
- popToRoute(route) - Pop to a particular scene, as specified by its route. All scenes after it will be unmounted
- popToTop() - Pop to the first scene in the stack, unmounting every other scene

最后调用AppReistry来注册JS入口：
```
React.AppRegistry.registerComponent('PushDemoApp', () => PushDemoApp);
```

我们再来看一下push_activity.js这个类，这是应用的启动界面，先来看一下render()方法，render()中return的内容相当于Android中的xml
布局，React Native使用了css-layout，实现了flexbox，即弹性盒布局。下面我们以push_activity.js中的界面为例一起来学一下这个布局。

> push_activity.js
```
render() {
        return (
            <ScrollView style = { styles.parent }>
            
            <Text style = { styles.textStyle }>
              { this.state.appkey }
            </Text>
            <Text style = { styles.textStyle }>
              { this.state.imei }
            </Text>
            <Text style  = { styles.textStyle }>
              { this.state.package }
            </Text>
            <Text style = { styles.textStyle }>
              { this.state.deviceId }
            </Text> 
            <Text style = { styles.textStyle }>
              { this.state.version }
            </Text>
            <TouchableHighlight
              underlayColor = '#0866d9'
              activeOpacity = { 0.5 }
              style = { styles.btnStyle }
              onPress = { this.jumpSetActivity }>
              <Text style = { styles.btnTextStyle }>
                设置
              </Text>
            </TouchableHighlight>
            <TouchableHighlight
              underlayColor = '#0866d9'
              activeOpacity = { 0.5 }
              style = { styles.btnStyle }
              onPress = { this.onInitPress }>
                <Text style = { styles.btnTextStyle }>
                  INITPUSH
                </Text>
            </TouchableHighlight>
            <TouchableHighlight
              underlayColor = '#e4083f'
              activeOpacity = { 0.5 }
              style = { styles.btnStyle }
              onPress = { this.onStopPress }>
                <Text style = { styles.btnTextStyle }>
                  STOPPUSH
                </Text>
            </TouchableHighlight>
            <TouchableHighlight
              underlayColor = '#f5a402'
              activeOpacity = { 0.5 }
              style = { styles.btnStyle }
              onPress = { this.onResumePress }>
                <Text style = { styles.btnTextStyle }> 
                  RESUMEPUSH
                </Text>
            </TouchableHighlight>
            <Text style = { styles.textStyle }>
              { this.state.pushMsg }
            </Text>
            </ScrollView>

          )
    }
    
  //style:
var styles = React.StyleSheet.create({
  parent: {
    padding: 15,
    backgroundColor: '#f0f1f3'
  },

  textStyle: {
    marginTop: 10,
    textAlign: 'center',
    fontSize: 20,
    color: '#808080'
  },

  btnStyle: {
    marginTop: 10,
    borderWidth: 1,
    borderColor: '#3e83d7',
    borderRadius: 8,
    backgroundColor: '#3e83d7'
  },
  btnTextStyle: {
    textAlign: 'center',
    fontSize: 25,
    color: '#ffffff'
  },
  inputStyle: {
    borderColor: '#48bbec',
    borderWidth: 1,

  },
});
```
Flexbox布局默认是线性布局即Android中的LinearLayout，PushActivity最外面由ScrollView包裹，style属性指定了该控件使用了哪种style，
如果一个控件没有声明高度或宽度，则默认填充满父布局。或者
```
container: {
  flex: 1,
}
```
flex:1这个也相当于match_parent。

flexDirection:指定排列方向，有row, column这两个值，分别是水平及垂直排列，默认为垂直排列。
alignItems与justifyContent这两个属性都指定控件在布局中的对齐方式，在垂直布局中，alignItem决定y轴方向，justifyContent决定x轴
方向，而在水平布局中则相反。如下图所示：



