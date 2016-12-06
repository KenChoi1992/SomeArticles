##在 Eclipse 上导入 JChat

##注意新版本用了 ActiveAndroid，需要手动导入 ActiveAndroid 的 jar（Android Studio 则不需要），[下载传送门](https://github.com/pardom/ActiveAndroid/downloads)

- 在官网或者在 github 下载 zip 后，解压(或者 git clone)；
- Import JChat：在 Eclipse 中选择 File -> Import -> Existing Android Project，然后点击 next

![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/import.PNG)

点击 Browser，找到你刚才下载并解压的目录
![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/import2.PNG)

->选择 JChat 文件夹，然后确定；

![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504112620.png)

- 修改 AndroidManifest，将“您的包名”和 AppKey 替换成自己在极光推送控制台上注册的包名和 AppKey；如果是从 github 上下载的压缩包，则只需要将 package 以及 ${applicationId} 替换成上述在极光控制台上注册的包名即可（如果没有可以[前往注册](https://www.jpush.cn/)）。

![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504112150.png)

- 全局替换 R 引用：选中该项目按 Ctrl + H（全局搜索） , 在弹出的对话框中选择 File Search --> 在 Containing text 中填写：
import io.jchat.android.R; （别忘了分号）然后点击 Replace
![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/replace.png)

在弹出的对话框中，全局替换：import io.jchat.android.R; 替换为 import 您的包名.R;
![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/replace2.png)

- 增加 support 包依赖，有以下三个步骤：
 1. 下载 Support Library；可以通过打开 SDK Manager 下载 extra 包来更新：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504112751.png) 

 2. Import 刚才你下载的 support-v7-appcompat 项目(选择../sdk/extras/android/support/v7/appcompat)：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/import4.PNG)

 如果这一步出现了错误：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/cuowu1.PNG)
 
 则可以将 support-v7-appcompat 下的 project.properties 中的 target 改为23（依据上一步中下载的 support 版本），JChat 中的 target 也改为23
 
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/project.properties.PNG)
 
 3. 在 JChat 中添加 support-v7-appcompat 依赖，右键点击 JChat 项目 -> 选择 properties
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504111420.png)

 在弹出的菜单的左侧列表中选择 Android，然后点击 Add 选项：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504111454.png)
 
 在弹出的菜单中选择 support-v7-appcompat 项目：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504111521.png)
 
 然后点击确定。如果出现如下错误：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/cuowu.png)
 
 可以在 JChat 的 Properties 菜单中选择 Java Build Path，然后在 Libraries 点击 Add JARs 选项，增加 support-v7-appcompat.jar：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/cuowu2.png)
 
完成上述的所有步骤后，即可选中 JChat 项目执行 Project -> Build Project，完成后运行即可。
