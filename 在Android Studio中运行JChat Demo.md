
####在Android Studio中导入JChat

1. 下载jchat.zip或者在[这里](https://www.jpush.cn/common/downloads/sdk/im_android/)下载.

2. 解压jchat.zip，在Android Studio中新建一个project或者在你当前的project中选择import module--> jchat（选择jchat文件夹）![如图](https://github.com/KenChoi1992/jchat-android/raw/dev/JChat/screenshots/screenshot2.png)

3. 修改jchat module下的build.gradle文件，将buildToolsVersion改为你Android Studio当前所使用的版本，sync一下。

4. 修改AndroidManifest，将Manifest中“您的包名”、“您的AppKey”全部替换为你在JPush控制台上注册应用的包名和对应的AppKey。（或者将“您的包名”全部替换成${applicationId}，然后在build.gradle文件中defaultConfig集合中声明一个applicationId。

![如图](https://github.com/KenChoi1992/jchat-android/raw/dev/JChat/screenshots/screenshot3.png)）

5. 全局替换R引用。选择src下io.jchat.android目录，右键点击，选择Replace in Path... 

![如图](https://github.com/KenChoi1992/jchat-android/raw/dev/JChat/screenshots/screenshot4.png)

在弹出的菜单中输入你要替换的R文件引用

![如图](https://github.com/KenChoi1992/jchat-android/raw/dev/JChat/screenshots/screenshot5.png)

然后点击find，完成后在菜单栏中选择Build-->Make Module 'JChat'即可。
