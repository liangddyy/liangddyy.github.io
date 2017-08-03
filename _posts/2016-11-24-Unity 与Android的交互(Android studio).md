---
layout: post
title: "Unity 与Android的交互(Android studio)"
keywords: 
description: ""
category: Unity Android
tags: Unity Android
---

<!--markdown-->主要介绍在Unity里调用Android原生代码，即用Androidstudio导出 aar 文件，在Unity里使用。源码在文末给出。  

我使用的环境：AndroidStudio2.2 +  Unity5.3

### Android部分  

#### 新建Android工程  
  
包名随意  
  
选择最低版本

![最低版本.png][1]  
  
我这里选择的是19，之后需要在Unity中修改为此包名和相同的最低版本。  

#### 复制Unity提供的jar包  

在Unity安装目录

 `Unity\Editor\Data\PlaybackEngines\AndroidPlayer\Variations\mono\Release\Classes` 下，复制 classes.jar 文件到Android项目的libs目录下：（注意切换到Project）  
  
![libs](http://539go.com/usr/uploads/2016/11-24/libs.png)  
  
左上角切换回Android工作目录方便操作  
  
#### 修改`build.gradle`文件  
  
![BaiduShurufa_2016-11-24_11-13-35](http://539go.com/usr/uploads/2016/11-24/BaiduShurufa_2016-11-24_11-13-35.png)  
  
在`build.gradle`文件（注意是app的build.gradle）里的dependencies 下做如下修改  
  
添加：（引用jar包）  
  
```  
compile files('libs/classes.jar')  
```  
  
删除：（删除不需要用的jar包）  
  
```  
compile 'com.android.support:appcompat-v7:25.0.0'  
testCompile 'junit:junit:4.12'  
```  
  
（此时 点Sync Now 编译会报错，因为通常新建的Android工程有对V7包的引用。暂时忽略，下面会讲解决办法）  
  
在`build.gradle`文件顶部 类似下面的代码中：  
  
```  
apply plugin: 'com.android.application'  
  
android {  
    compileSdkVersion 25  
    buildToolsVersion "25.0.0"  
    defaultConfig {  
        applicationId "com.example.unitydemo_android"  
```  
  
需要将 
  
`apply plugin: 'com.android.application' `修改为  
  
`apply plugin: 'com.android.library'`  
  
并注释掉  
  
```  
applicationId "com.example.unitydemo_android"  
```  
  
#### 修改 MainActivity.java 文件  
  
先将类的继承修改为继承`UnityPlayerActivity`， 代码如下：  
```  
public class MainActivity extends UnityPlayerActivity {  
```  
  
然后注释掉这一行：（否则会在Unity中启动Android布局）  
  
```  
setContentView(R.layout.activity_main);  
```  
  
并添加一个方法用于在Unity中调用：  
  
```  
public void ShowToast(final String msg){  
    // 需要在UI线程下执行  
    runOnUiThread(new Runnable() {  
        @Override  
        public void run() {  
            Toast.makeText(getApplicationContext(),msg,Toast.LENGTH_LONG).show();  
        }  
    });  
}  
```  
  
> 坑点  
  
上面的Toast显示 需要在UI线程下执行。否则在Unity中看不到。  
  
#### 修改`AndroidManifest.xml`文件  
  
![444](http://539go.com/usr/uploads/2016/11-24/444.png)  
  
在AndroidManifest.xml中添加这一行  
  
```  
<meta-data android:name="unityplayer.UnityActivity" android:value="true" />  
```  
  
AndroidManifest.xml 文件中可以看到主题引用的是Style中的设置`android:theme="@style/AppTheme"`，跳转到 Style.xml文件可以看到类下面的代码：  
  
```  
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">  
    <!-- Customize your theme here. -->  
    <item name="colorPrimary">@color/colorPrimary</item>  
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>  
    <item name="colorAccent">@color/colorAccent</item>  
</style>  
```  
  
修改为：  
  
```  
<style name="AppTheme" parent="android:Theme.Light"></style>  
```  
  
目的是为了去掉对V7包的引用。  
  
此时重新 Sync Now ,已经不会报错。然后 Build APK。  
  
#### 删除多余的引用  
  
Build 完以后，` \app\build\outputs\aarapp-debug.aar`  文件，用压缩软件打开并删除掉 \libs\classes.jar 文件。（避免重复引用）  
  
### Unity部分  
  
#### 新建Unity工程  
  
将Android工程下的这两个文件：  
  
```  
\app\build\outputs\aarapp-debug.aar  
\app\src\main\AndroidManifest.xml  
```  
  
复制到 Assets 下的 `Plugins\Android\` 目录下  
  
![123](http://539go.com/usr/uploads/2016/11-24/123.png)  
  
新建一个按钮并绑定下面的脚本用于测试：  
  
    public void BtnShwMessage()  
    {  
        //AndroidJavaClass：通过指定类名可以构造出一个类  
        AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");  
    
    	// UnityPlayer这个类可以获取当前的Activity  
        // currentActivity字符串对应源码中UnityPlayer类下 的 Activity 变量名。  
        AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject>("currentActivity");  
        
        // 在对象上调用一个Java方法  
        jo.Call("ShowToast", "Unity 调用了这个方法");  
    }  
这几个方法用途可以参见官方的API。  
  
#### 切换平台并配置  
  
切换到Android平台：  
  
​	File - BuildSettings - Switch Platform（Android）  
  
配置：  
  
在 Player Setting 里，修改下面两项：![BaiduShurufa_2016-11-24_11-47-32](http://539go.com/usr/uploads/2016/11-24/BaiduShurufa_2016-11-24_11-47-32.png)  
  
Bundle Identifier 需修改为 Android端 AndroidManifest.xml 文件里的package 里的内容  
  
最低的Api版本设置为 新建Android工程时所选的最低Api版本。这里是19。  
  
然后，在Unity里 Build & Run吧。  
  
#### 运行  
  
点击按钮 截图  
  
![324](http://539go.com/usr/uploads/2016/11-24/324.png)  
  
#### 源码  
  
链接：[http://pan.baidu.com/s/1skFq24X][2] 密码：p33w  
  
  
  [1]: http://539go.com/usr/uploads/2016/11/3165343850.png  
  [2]: http://pan.baidu.com/s/1skFq24X  
