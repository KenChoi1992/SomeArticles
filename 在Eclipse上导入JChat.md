##在Eclipse上导入JChat

- 在官网或者在github下载zip后，解压；
- 在Eclipse中选择File-> Import -> Existing Android Project ->你刚才下载并解压的目录->选择JChat文件夹，然后确定；
- 修改AndroidManifest，将“您的报名”和AppKey替换成自己在极光推送控制台上注册的包名和AppKey；
- 全局替换R引用：选中该项目按Ctrl + H（全局搜索） , 在弹出的对话框中选择File Search-->在Containing text中填写：
import io.jchat.android.R; （别忘了分号）然后点击Replace在弹出的对话框中，全局替换：import io.jchat.android.R;替换为import 您的包名.R;
- 增加support包依赖，有以下三个步骤：
 1. 下载Support Library；
 2. 从Android SDK目录下的extra文件夹中找到support－v4 Jar包，并拷贝到你的libs下；
 3. 创建一个基于Support Library的library project，然后在你的项目的properties中将android-support-v7-appcompat添加到library中
