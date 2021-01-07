---
title: Android接入文档（国内）
author: wuxiaowei
date: 2020-12-22 15:00:00 +0800
categories: [Blogging, Tutorial]
tags: [Android,国内]
pin: true
---

## 迁移到 AndroidX

如果您的项目已经是支持Androidx，请忽略。

### 使用 Android Studio 迁移现有项目

使用 Android Studio 3.2 及更高版本，您只需从菜单栏中依次选择 Refactor > Migrate to AndroidX，即可将现有项目迁移到 AndroidX。

重构命令使用两个标记。默认情况下，这两个标记在 gradle.properties 文件中都设为 true：

> android.useAndroidX=true
       
Android 插件会使用对应的 AndroidX 库而非支持库。
> android.enableJetifier=true 

Android 插件会通过重写现有第三方库的二进制文件，自动将这些库迁移为使用 AndroidX，**特别需要注意原生广告**。

还有疑问可以看google的[**迁移到 AndroidX**](https://developer.android.com/jetpack/androidx/migrate)文档


## MultiDex 配置

### minSdkVersion >=21

系统默认情况下会启用 MultiDex，并且您不需要 MultiDex 支持库。

### minSdkVersion <=20

您必须使用 MultiDex 支持库并对应用项目进行以下修改：

### 修改模块级 build.gradle 文件

启用 MultiDex，并将 MultiDex 库添加为依赖项，如下所示：

    ```gradle
    android {
        defaultConfig {
            ...
            multiDexEnabled true
        }
        ...
    }

    dependencies {
        implementation 'androidx.multidex:multidex:2.0.1'
    }
    ```

### 继承 Application 类，执行以下某项操作：

+ 重写 attachBaseContext() 方法并调用 MultiDex.install(this) 以启用 MultiDex：
  ```java
    public class MyApplication extends Application {
        @Override
        protected void attachBaseContext(Context base) {
            super.attachBaseContext(base);
            MultiDex.install(this);
        }
    }
  ```
+ 请修改manifest文件以设置 <application> 标记中的 android:name，替换成你的Application全类名
  
  ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="你的包名">
        <application
                android:name="com.xxx.MyApplication" >
            ...
        </application>
    </manifest>

  ```
[更多MultiDex相关文档](https://developer.android.com/studio/build/multidex)    


## 项目配置修改

### 配置google-services.json

从firebase控制台下载 google-services.json ，并复制到module根目录下

### 项目根目录的build.gradle增加以下内容

```groovy
buildscript {

    repositories {
        maven { url 'https://dl.bintray.com/umsdk/release' }
    }

    allprojects {
        repositories {
            
            maven {
                url 'https://repo.rdc.aliyun.com/repository/74503-release-qNEqtU/'
                credentials {
                    username 'p5gXfa'
                    password 'wKY0RHNSH3'
                }
            }
            maven { url 'https://dl.bintray.com/umsdk/release' }
        
        }
    }
}
```

### 修改app module的build.gradle 

添加以下内容

```groovy

repositories {
    flatDir {
        dirs 'libs'
    }
}
dependencies {
    //sdk核心库（必须）
    implementation 'com.eyu.opensdk:core-ch:1.7.21'

    //按需求引入广告平台
    //mtg
    //implementation 'com.eyu.opensdk.ad.mediation:mtg-ch-adapter:13.0.41.21'
    //穿山甲
    //implementation 'com.eyu.opensdk.ad.mediation:pangle-ch-adapter:3.3.0.3.21'
    //广点通
    //implementation 'com.eyu.opensdk.ad.mediation:gdt-adapter:4.294.1164.21'
     //topon
    //implementation 'com.eyu.opensdk.ad.mediation:topon-adapter:5.7.3.21'
}
```

### 穿山甲的库需要单独引入

将穿山甲的库拷贝到工程目录的libs下，头条库open_ad_sdk.aar在这里[app_ch_new](https://github.com/EyugameQy/EyuLibrary-android/tree/master/app_ch_new/libs)，在gradle中加入

```groovy
dependencies {
    implementation(name: 'open_ad_sdk', ext: "aar")
}

```

### topon的库需要单独引入

将topon的库拷贝到工程目录的libs下，库在这里[app_ch_new](https://github.com/EyugameQy/EyuLibrary-android/tree/master/app_ch_new/libs/)，topon_libs和topon_res整个**文件夹**复制到工程目录的libs，然后在gradle中的android下加入以下内容

```groovy
android{
    sourceSets {
        main {
            res.srcDirs += 'topon_res'
        }
    }
}

dependencies {
    api fileTree(include: ['*.jar','*.aar'], dir: 'topon_libs')
}

```

### 清单文件修改

#### Android 9以上适配

+ 在AndroidManifest中新增以下配置
  
```xml
<application>
    
    <uses-library android:name="org.apache.http.legacy" android:required="false"/>
    
</application>
```

+ 兼容部分第三方广告SDK存在Http请求 在AndroidManifest的application的标签中增加：android:networkSecurityConfig 的配置：

```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    
    >
    
</application>
```
其中在项目的res/xml文件夹新增network_security_config.xml，内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

+ 引入了哪个平台就加入哪个，不然会编译不通过，如果引入聚合，聚合中包含了以下平台，也需要加入。如果你是在库工程引入，需要吧${applicationId}替换成你的包名

```xml
<!--头条-穿山甲-->
<provider
    android:name="com.bytedance.sdk.openadsdk.TTFileProvider"
    android:authorities="${applicationId}.TTFileProvider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/eyu_tt_file_path" />
</provider>
<provider
    android:name="com.bytedance.sdk.openadsdk.multipro.TTMultiProvider"   
    android:authorities="${applicationId}.TTMultiProvider"   
    android:exported="false" />
<!--mtg-->
<provider
    android:name="com.mintegral.msdk.base.utils.MTGFileProvider"
    android:authorities="${applicationId}.mtgFileProvider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/eyu_mtg_file_path"/>
</provider>
<!--广点通-->
<provider
    android:name="com.qq.e.comm.GDTFileProvider"
    android:authorities="${applicationId}.gdt.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
     android:name="android.support.FILE_PROVIDER_PATHS"
     android:resource="@xml/eyu_gdt_file_path" />
</provider>
```


## SDK使用

### sdk初始化

请在 Application 中初始化sdk，添加配置信息，按需添加

```java
//
InitializerBuilderImpl builder = new InitializerBuilderImpl();

//appsflyer配置
//builder.initAppsFlyer(“appkey”);
//热云
//builder.initTracking(this,"appKey","channle");
//友盟
//builder.initUmeng("appKey","channle");
//数数的统计初始化
//builder.initThinkData("appid",BuildConfig.DEBUG);

SdkCompat.getInstance().init(Application, builder);

```

### 广告配置与初始化

#### 广告配置

广告配置有三个文件，ad_setting.json，ad_cache_setting.json，ad_key_setting.json
+ ad_setting.json，广告位配置，展示广告传入的 <font color = #1a73e8 size=3>adPlaceId</font> 就是id的值，格式如下：
    ```json
    [
        {
        "cacheGroup": "main_view_inter_ad",
        "isEnabled": "true",
        "nativeAdLayout": "",
        "id": "main_view_inter_ad",
        "desc": "首页插屏"
        }
    ]
    ```
+ ad_cache_setting.json，广告的缓存池配置
    ```json
    {
        "keys": "[\"fb_ys_a\",\"adys_sy\"]",//广告平台key
        "isAutoLoad": "true",
        "id": "page_view_native_ad",
        "type": "nativeAd"//广告类型
    }
    ```
+ ad_key_setting.json，广告平台的key
    ```json
    [
        {"id":"adcp_js","key":"ca-app-pub-3940256099942544/1033173712","network":"admob"},
        {"id":"adjl_jsg","key":"ca-app-pub-3940256099942544/5224354917","network":"admob"},
        {"id":"adys_sy","key":"ca-app-pub-3940256099942544/2247696110","network":"admob"},
        {"id":"adys_ba","key":"ca-app-pub-3940256099942544/6300978111","network":"admob"}
    ]
    ```


#### 权限申请

```java
 String[] permissions = {Manifest.permission.READ_PHONE_STATE, Manifest.permission.WRITE_EXTERNAL_STORAGE,
                Manifest.permission.ACCESS_FINE_LOCATION};
SdkCompat.getInstance().requestPermissions(this, permissions, 1000);
```

#### 权限回调

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    SdkCompat.getInstance().onRequestPermissionsResult(this, requestCode, permissions, grantResults);
}
```

#### 广告初始化

```java
AdConfig adConfig = new AdConfig();
adConfig.setAdPlaceConfigResource(this, R.raw.ad_setting);
//adConfig.setAdPlaceConfigStr

adConfig.setAdKeyConfigResource(this, R.raw.ad_key_setting);
//adConfig.setAdKeyConfigStr()

adConfig.setAdGroupConfigResource(this, R.raw.ad_cache_setting);
//adConfig.setAdGroupConfigStr

//穿山甲
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
bundle.putString(PangleExtras.APP_NAME, "");
adConfig.addPlatformConfig(AdPlatform.PANGLE, bundle);

//mtg
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
bundle.putString(PlatformExtras.COMMON_APP_KEY, "");
adConfig.addPlatformConfig(AdPlatform.MTG, bundle);

//TOPON
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
bundle.putString(PlatformExtras.COMMON_APP_KEY, "");
adConfig.addPlatformConfig(AdPlatform.TOPON, bundle);

//广点通
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
adConfig.addPlatformConfig(AdPlatform.GDT, bundle);


EyuAdManager.getInstance().config(MainActivity.this, adConfig, new EyuAdsListener() {

    @Override
    public void onAdReward(AdFormat type, String placeId) {
        //激励视频获得奖励
    }

    @Override
    public void onAdLoaded(AdFormat type, String placeId) {
        //广告加载成功
    }

    @Override
    public void onAdShowed(AdFormat type, String placeId) {
        //广告展示
    }

    @Override
    public void onAdClosed(AdFormat type, String placeId) {
        //广告被关闭
    }

    @Override
    public void onAdClicked(AdFormat type, String placeId) {
        //广告被点击
    }

    @Override
    public void onDefaultNativeAdClicked() {

    }

    @Override
    public void onAdLoadFailed(AdFormat type, String placeId, String key, int code) {
        //广告加载失败，如果没有广告展示，可以过滤日志onAdLoadFailed查看广告加载失败原因
    }

    @Override
    public void onImpression(AdFormat type, String placeId) {
        //广告展示，同onAdShowed
    }

});
   
```

### 广告使用示例

调用show方法时传入的<font color = #1a73e8 size=3>adPlaceId</font>为4.2.1中的<font color = #1a73e8 size=3>ad_setting.json</font>中的id

#### 判断广告是否有可用的广告

```java
EyuAdManager.getInstance().isAdLoaded(AdFormat,"adPlaceId")
```

#### 展示激励视频

```java
EyuAdManager.getInstance().show(AdFormat.REWARDED, Activity,"adPlaceId");
```

#### 展示插屏

```java
EyuAdManager.getInstance().show(AdFormat.INTERSTITIAL, Activity,"adPlaceId");
```

#### 展示banner

需要传入一个ViewGroup，这个group是用来放banner的
```java
EyuAdManager.getInstance().show(AdFormat.BANNER, Activity,ViewGroup,"adPlaceId");
```

#### 展示原生广告

需要传入一个ViewGroup，这个group是用来放native的
```java
EyuAdManager.getInstance().show(AdFormat.NATIVE, Activity,ViewGroup,"adPlaceId");

```


## 事件埋点

### 基本数据埋点

调用下面的方法，事件会上传到umeng和Appsflyer，**不会上传到数数** 如果要上传到数数，请额外调用数数的方法
```java
//事件不带参数
EventHelper.getInstance().logEvent("事件名称");
//事件带map参数
EventHelper.getInstance().logEventWithParamsMap("事件名称",new HashMap<String, Object>());
//事件带json字符串参数
EventHelper.getInstance().logEventWithJsonParams("事件名称","json");
```

### 数数的埋点

EventHelper对数数sdk的方法只是做了一层简单的封装，并没有做任何处理，所以先看下[数数的文档](https://docs.thinkingdata.cn/ta-manual/latest/installation/installation_menu/client_sdk/android_sdk_installation/android_sdk_installation.html#%E4%B8%89%E3%80%81%E5%8F%91%E9%80%81%E4%BA%8B%E4%BB%B6)，了解每个方法的含义


```java

EventHelper.getInstance().track("事件名称");

EventHelper.getInstance().track("事件名称",JSONObject);
//....
void track(String var1);

void track(String var1, JSONObject var2);

void timeEvent(String var1);

void login(String var1);

void logout();

void identify(String var1);

void user_set(JSONObject var1);

void user_setOnce(JSONObject var1);

void user_add(JSONObject var1);

void user_append(JSONObject var1);

void user_add(String var1, Number var2);

void user_delete();

void user_unset(String... var1);

void setSuperProperties(JSONObject var1);

void trackAppInstall();

void trackFirst(String var1, JSONObject var2);

void trackUpdate(String var1, JSONObject var2, String var3);
```

## 常见问题

+ sdk下载失败？  
  检查build.gradle配置是否添加，如果添加好后还是不能加载成功，请检查网络是否连通

+ 没有广告展示？  
  1.请检查广告配置是否正确配置，如果配置好了，在Android studio的日志打印那里过滤onAdLoadFailed，有错误码打印，将错误码提供给支持
  2.确保科学上网  
  3.按照6中广告测试方法
  4.Facebook广告必须安装Facebook且登录账号


## 示例工程 

[示例工程](https://github.com/EyugameQy/EyuLibrary-android/tree/master/app_ch_new)，建议先仔细看一遍上面的文档
