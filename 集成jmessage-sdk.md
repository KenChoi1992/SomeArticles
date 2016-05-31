####在你的项目集中集成 jmessage-sdk

* 类库配置

在下载的 JChat demo 中打开 libs 文件夹，将 libs 的 so 库文件以及 jmessage-sdk 拷贝到你的项目中，目录结构![如图](https://github.com/KenChoi1992/jchat-android/raw/dev/JChat/screenshots/screenshot1.png)

接下来，修改你项目中的 build.gradle 文件，在 android 块中加入 sourceSets
> demo build.gradle

```
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            jniLibs.srcDirs = ['libs']
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        // Move the tests to tests/java, tests/res, etc...
        instrumentTest.setRoot('tests')

        // Move the build types to build-types/<type>
        // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...
        // This moves them out of them default location under src/<type>/... which would
        // conflict with src/ being used by the main source set.
        // Adding new build types or product flavors should be accompanied
        // by a similar customization.
        debug.setRoot('build-types/debug')
        release.setRoot('build-types/release')
    }
```
这样可以兼容 Android Studio 和 Eclipse。

* AndroidManifest 配置

在 demo 中将 jmessage-sdk 以及 jpush 需求的配置项复制过来（jmessage 集成了 jpush 的功能）

权限声明
> demo AndroidManifest.xml

```

    <permission
        android:name="${applicationId}.permission.JPUSH_MESSAGE"
        android:protectionLevel="signature" />

    <!--Required 一些系统要求的权限，如访问网络等-->
    <uses-permission android:name="${applicationId}.permission.JPUSH_MESSAGE" />
    <uses-permission android:name="android.permission.RECEIVE_USER_PRESENT" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_LOCATION_EXTRA_COMMANDS"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>


    <!-- JMessage Demo required for record audio-->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>

```

application 配置项
> demo AndroidManifest.xml

```

        <!-- Required Push SDK核心功能-->
        <service
            android:name="cn.jpush.android.service.PushService"
            android:enabled="true"
            android:exported="false"
            android:process=":remote">
            <intent-filter>
                <action android:name="cn.jpush.android.intent.REGISTER" />
                <action android:name="cn.jpush.android.intent.REPORT" />
                <action android:name="cn.jpush.android.intent.PushService" />
                <action android:name="cn.jpush.android.intent.PUSH_TIME" />
            </intent-filter>
        </service>

        <!-- Required Push SDK核心功能-->
        <receiver
            android:name="cn.jpush.android.service.PushReceiver"
            android:enabled="true"
            android:exported="false">
            <intent-filter android:priority="1000">
                <action android:name="cn.jpush.android.intent.NOTIFICATION_RECEIVED_PROXY" />  <!--Required  显示通知栏 -->
                <category android:name="${applicationId}" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.USER_PRESENT" />
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            </intent-filter>
            <!-- Optional -->
            <intent-filter>
                <action android:name="android.intent.action.PACKAGE_ADDED" />
                <action android:name="android.intent.action.PACKAGE_REMOVED" />

                <data android:scheme="package" />
            </intent-filter>
        </receiver>

        <!-- Required Push SDK核心功能 -->
        <activity
            android:name="cn.jpush.android.ui.PushActivity"
            android:configChanges="orientation|keyboardHidden"
            android:theme="@android:style/Theme.NoTitleBar"
            android:exported="false">
            <intent-filter>
                <action android:name="cn.jpush.android.ui.PushActivity" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="${applicationId}" />
            </intent-filter>
        </activity>
        <!-- Required Push SDK核心功能 -->
        <service
            android:name="cn.jpush.android.service.DownloadService"
            android:enabled="true"
            android:exported="false" />
        <!-- Required Push SDK核心功能 -->
        <receiver android:name="cn.jpush.android.service.AlarmReceiver" android:exported="false"/>

        <!-- IM Required IM SDK核心功能-->
        <receiver
            android:name="cn.jpush.im.android.helpers.IMReceiver"
            android:enabled="true"
            android:exported="false">
            <intent-filter android:priority="1000">
                <action android:name="cn.jpush.im.android.action.IM_RESPONSE" />
                <action android:name="cn.jpush.im.android.action.NOTIFICATION_CLICK_PROXY" />

                <category android:name="${applicationId}" />
            </intent-filter>
        </receiver>

        <!-- option 可选项。用于同一设备中不同应用的JPush服务相互拉起的功能。 -->
        <!-- 若不启用该功能可删除该组件，将不拉起其他应用也不能被其他应用拉起 -->
        <service
            android:name="cn.jpush.android.service.DaemonService"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="cn.jpush.android.intent.DaemonService" />
                <category android:name="${applicationId}" />
            </intent-filter>

        </service>

        <!-- Required. Enable it you can get statistics data with channel -->
        <meta-data
            android:name="JPUSH_CHANNEL"
            android:value="developer-default" />
        <!-- Required. AppKey copied from Portal -->
        <meta-data
            android:name="JPUSH_APPKEY"
            android:value="4f7aef34fb361292c566a1cd" /><!--  </>值来自开发者平台取得的AppKey-->

```

* 初始化 jmessage-sdk

在你的 application 类中，需要调用以下方法以初始化 jmessage-sdk
> demo JChatApplication.java

```
  JMessageClient.init(getApplicationContext());
  SharePreferenceManager.init(getApplicationContext(), JCHAT_CONFIGS);
  JMessageClient.setNotificationMode(JMessageClient.NOTI_MODE_DEFAULT);
  new NotificationClickEventReceiver(getApplicationContext());
```
如果你想自定义 Notification 的样式，可以将上面的 NotificationMode 的值设为 No_Notification，并且去掉下面的 NotificationClickEventReceiver，然后在接收到消息时再创建 Notification（下面会说到）。

在你的启动 Activity 的 onPause()和 onResume()方法中需要分别调用

```
JPushInterface.onPause(this);
```
以及
```
JPushInterface.onResume(this);
```
