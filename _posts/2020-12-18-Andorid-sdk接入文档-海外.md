---
title: Android接入文档（国外）
author: wuxiaowei
date: 2021-01-07 11:00:00 +0800
categories: [Blogging, Tutorial]
tags: [Android,海外]
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

### 修改模块级 build.gradle 文件以启用 MultiDex，并将 MultiDex 库添加为依赖项，如下所示：

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
       maven {
           url  "https://dl.bintray.com/mintegral-official/mintegral_ad_sdk_android_for_oversea"
       }
        maven { url "https://fyber.bintray.com/marketplace" }
        maven { url "https://dl.bintray.com/ironsource-mobile/android-sdk" }
    }
    dependencies {
        classpath 'com.google.gms:google-services:4.2.0'
        classpath 'com.google.firebase:firebase-crashlytics-gradle:2.4.1'
        
    }
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
        maven { url "https://jitpack.io" }
         maven {
             url  "https://dl.bintray.com/mintegral-official/mintegral_ad_sdk_android_for_oversea"
         }
        maven { url "https://fyber.bintray.com/marketplace" }
        maven { url "https://dl.bintray.com/ironsource-mobile/android-sdk" }
    }
}
```
### 从旧版升级的注意

删除旧版的引入，refresh gradle
```groovy
implementation 'com.eyu:eyulibrary:xxx'
```
新版的包名改了，将代码中的报错import删除，重新引入。删除SdkHelper相关代码

### app module的build.gradle 添加以下内容

```groovy
apply plugin: 'com.google.gms.google-services'
apply plugin: 'com.google.firebase.crashlytics'

dependencies {
    //删除旧版的引入
    //implementation 'com.eyu:eyulibrary:xxx'

    //sdk核心库（必须）
    implementation 'com.eyu.opensdk:core:1.7.21'
    
    
    //按需求引入广告平台
    //admob    
    //implementation 'com.eyu.opensdk.ad.mediation:admob-adapter:19.6.0.21'

    //admob聚合
    //implementation 'com.eyu.opensdk.ad.mediation:admob-compat_adapter:19.6.0.21'
    
    //max
    //implementation 'com.eyu.opensdk.ad.mediation:max-adapter:9.14.11.21'
    
    //facebook
    //implementation 'com.eyu.opensdk.ad.mediation:facebook-adapter:6.2.0.21'
    
    //applovin
    //implementation 'com.eyu.opensdk.ad.mediation:applovin-adapter:9.14.11.21'
    
    //mtg
    //implementation 'com.eyu.opensdk.ad.mediation:mtg-adapter:15.2.41.21'
    
    //穿山甲
    //implementation 'com.eyu.opensdk.ad.mediation:pangle-adapter:3.4.0.0.21'
    
    //unity
    //implementation 'com.eyu.opensdk.ad.mediation:unity-adapter:3.4.8.21'
    
    //vungle
    //implementation 'com.eyu.opensdk.ad.mediation:vungle-adapter:6.8.1.21'

    //tradplus
    //implementation 'com.eyu.opensdk.ad.mediation:tradplus-adapter:5.2.8.1.21'
}
```

### 清单文件修改

```xml
<manifest>
    <application>
        <!--google ads-->
       <meta-data
            android:name="com.google.android.gms.ads.APPLICATION_ID"
            android:value="@string/google_ads_app_id" />
        <!-- facebook ads-->
        <meta-data
            android:name="com.facebook.sdk.ApplicationId"
            android:value="@string/facebook_app_id" />
        <!-- applovin max ads-->
        <meta-data
            android:name="applovin.sdk.key"
            android:value="@string/applovin_sdk_key" />
        <!-- 穿山甲，如果你是在库工程引入，需要吧${applicationId}替换成你的包名-->
        <provider
            android:name="com.bytedance.sdk.openadsdk.multipro.TTMultiProvider"
            android:authorities="${applicationId}.TTMultiProvider"
            android:exported="false" />
    </application>
</manifest>
```

## SDK使用

### sdk初始化

请在 Application 中初始化sdk，添加配置信息，按需添加

```java
//
InitializerBuilderImpl builder = new InitializerBuilderImpl();

//appsflyer配置
//builder.initAppsFlyer(“appkey”);

//数数的统计初始化
//builder.initThinkData("appid","serverurl");

//远程配置
//Map<String, Object> defaultsMap = new HashMap<>();
//defaultsMap.put("key","defaultValue");
//builder.initRemoteConfig(sDefaultsMap);

SdkCompat.getInstance().init(Application, builder);

```

### 广告配置与初始化

#### 广告配置

广告配置有三个文件，ad_setting.json，ad_cache_setting.json，ad_key_setting.json
+ ad_setting.json，广告位配置，展示广告传入的 <font color = #1a73e8 size=3>adPlaceId</font> 就是id的值，示例：
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
+ ad_cache_setting.json，广告的缓存池配置，示例：
    ```json
    {
        "keys": "[\"fb_ys_a\",\"adys_sy\"]",//广告平台key
        "isAutoLoad": "true",
        "id": "page_view_native_ad",
        "type": "nativeAd"//广告类型
    }
    ```
+ ad_key_setting.json，广告平台的key，示例：
    ```json
    [
        {"id":"adcp_js","key":"ca-app-pub-3940256099942544/1033173712","network":"admob"},
        {"id":"adjl_jsg","key":"ca-app-pub-3940256099942544/5224354917","network":"admob"},
        {"id":"adys_sy","key":"ca-app-pub-3940256099942544/2247696110","network":"admob"},
        {"id":"adys_ba","key":"ca-app-pub-3940256099942544/6300978111","network":"admob"}
    ]
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

 //admob
Bundle bundle = new Bundle();
//appid 在Manifest中配置
bundle.putStringArrayList(PlatformExtras.COMMON_TEST_DEVICE, new ArrayList<String>(Arrays.asList("")));
adConfig.addPlatformConfig(AdPlatform.ADMOB, bundle);

//facebook
bundle = new Bundle();
//appid 在Manifest中配置
bundle.putString(PlatformExtras.COMMON_TEST_DEVICE, "");
adConfig.addPlatformConfig(AdPlatform.FACEBOOK, bundle);

//max
//appid 在Manifest中配置

//穿山甲
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
bundle.putString(PangleExtras.APP_NAME, "");
adConfig.addPlatformConfig(AdPlatform.PANGLE, bundle);

//unity
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
adConfig.addPlatformConfig(AdPlatform.UNITY, bundle);

//vungle
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
adConfig.addPlatformConfig(AdPlatform.VUNGLE, bundle);

//mtg
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
bundle.putString(PlatformExtras.COMMON_APP_KEY, "");
adConfig.addPlatformConfig(AdPlatform.MTG, bundle);

//TRADPLUS
bundle = new Bundle();
bundle.putString(PlatformExtras.COMMON_APP_ID, "");
adConfig.addPlatformConfig(AdPlatform.TRADPLUS, bundle);

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

调用show方法时传入的**adPlaceId** 为**ad_setting.json**中的id

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

调用下面的方法，事件会上传到Firebase和Appsflyer
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

## 广告测试

**强烈建议使用VPN挂到美国测试**，没有广告请检查日志打印，过滤onAdLoadFailed,一般失都是广告没有填充

### 谷歌

+ 使用示例广告单元  

    |  广告格式   | 示例广告单元 ID  |
    |  ----  | ----  |
    | 横幅广告  | ca-app-pub-3940256099942544/6300978111 |
    | 插页式广告  | ca-app-pub-3940256099942544/1033173712 |
    | 插页式视频广告  | ca-app-pub-3940256099942544/8691691433 |
    | 激励视频广告  | ca-app-pub-3940256099942544/5224354917 |
    | 原生高级广告  | ca-app-pub-3940256099942544/2247696110 |
    | 原生高级视频广告  | ca-app-pub-3940256099942544/1044960115 |  

<br>

+ 使用测试设备  
    >系统会自动将 Android 模拟器配置为测试设备。
+ 检查 logcat 输出，过滤addTestDevice查找设备id，将设备id加入的初始化的代码当中 
    ```
    I/Ads: Use AdRequest.Builder.addTestDevice("68F0142924806103623C22CBA2697DB1") to get test ads on this device.
    ```
 


### Facebook

检测logcat输出，过滤"Test mode device hash"，将其添加到初始化配置


## 常见问题

+ sdk下载失败？  
  检查build.gradle配置是否添加，如果添加好后还是不能加载成功，请检查网络是否连通

+ 没有广告展示？  
  1.请检查广告配置是否正确配置，如果配置好了，在Android studio的日志打印那里过滤onAdLoadFailed，有错误码打印
  2.确保科学上网  
  3.按照6中广告测试方法
  4.Facebook广告必须安装Facebook且登录账号

+ 错误码的含义？  
  Admob错误码  
  |  错误码   | 含义  |  
    |  ----  | ----  |
    | 0  | 内部错误. |
    | 1  | 请求参数错误，例如广告key错误 |
    | 2  | 网络异常，请求失败. |
    | 3  | 没有广告填充. |
    | 9  | 聚合广告没有广告填充. |

    Facebook错误码  
    |  错误码   | 含义  |
    |  ----  | ----  |
    | 1000  | Network Error. |
    | 1001  | 没有广告填充，必须安装Facebook且登录 |
    | 1002  | 广告加载太频繁. |
    | 1012  | 广告sdk版本太低. |
    | 2000，2001  | 内部错误. |

    。。。。待完善

## 示例工程 
[示例工程](https://github.com/EyugameQy/EyuLibrary-android/tree/master/app_overseas_new)，建议先仔细看一遍上面的文档
