##在Eclipse上导入JChat

- 在官网或者在github下载zip后，解压；
- 在Eclipse中选择File-> Import -> Existing Android Project，然后点击next

![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/import.PNG)

点击Browser，找到你刚才下载并解压的目录
![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/import2.PNG)

->选择JChat文件夹，然后确定；

![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504112620.png)

- 修改AndroidManifest，将“您的报名”和AppKey替换成自己在极光推送控制台上注册的包名和AppKey；如果是从github上下载的压缩包，则只需要将package以及${applicationId}替换成上述在极光控制台上注册的包名即可（如果没有可以[前往注册](https://www.jpush.cn/)）。

![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504112150.png)

- 全局替换R引用：选中该项目按Ctrl + H（全局搜索） , 在弹出的对话框中选择File Search-->在Containing text中填写：
import io.jchat.android.R; （别忘了分号）然后点击Replace
![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/replace.png)

在弹出的对话框中，全局替换：import io.jchat.android.R;替换为import 您的包名.R;
![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/replace2.png)

- 增加support包依赖，有以下三个步骤：
 1. 下载Support Library；可以通过打开SDK Manager下载extra包来更新：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504112751.png) 

 2. import刚才你下载的support-v7-appcompat项目(选择../sdk/extras/android/support/v7/appcompat)：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/import4.PNG)

 如果这一步出现了错误：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/cuowu1.PNG)
 
 则可以将support-v7-appcompat下的project.properties改为23（依据上一步中下载的support版本）
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/project.properties.PNG)
 
 3. 在JChat中添加support-v7-appcompat依赖，右键点击JChat项目->选择properties
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504111420.png)

 在弹出的菜单的左侧列表中选择Android，然后点击Add选项：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504111454.png)
 
 在弹出的菜单中选择support-v7-appcompat项目：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/QQ%CD%BC%C6%AC20160504111521.png)
 
 然后点击确定。如果出现如下错误：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/cuowu.png)
 
 可以在JChat的Properties菜单中选择Java Build Path，然后在Libraries点击Add JARs选项，增加support-v7-appcompat.jar：
 ![](https://github.com/KenChoi1992/SomeArticles/blob/master/screenshots/cuowu2.png)
 
完成上述的所有步骤后，即可选中JChat项目执行Project->Build Project，完成后运行即可。
