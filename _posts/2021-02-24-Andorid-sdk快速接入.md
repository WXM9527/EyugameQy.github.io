---
title: Android快速接入
author: wuxiaowei
date: 2021-03-18 18:00:00 +0800
categories: [Blogging, Tutorial]
tags: [Android,快速接入]
pin: true
---

# 快速接入
快速接入是通过引入提供的库工程，该库工程是对sdk的一个封装与集成，旨在降低接入成本，适用于对Android不太熟的开发者。<br>
将[库工程](https://github.com/EyugameQy/EyuLibrary-android/)下载到本地，然后在android studio中File->New->import module，将**quick_start**这个module引入到你自己的项目中

# 基础配置
针对你的工程
## AndroidX配置

修改根目录的gradle.properties，加入以下代码块

```
android.useAndroidX=true
android.enableJetifier=true 

```


## 修改phone module（运行的那个）下的build.gradle 文件

启用 MultiDex，并将 MultiDex 库添加为依赖项，如下所示：

```groovy
    android {
        defaultConfig {
            ...
            multiDexEnabled true
        }
        ...
    }

```


## （国外）Firebase配置

从firebase控制台下载 google-services.json ，并复制到 Phone Module目录下


## （国外）Phone Module的build.gradle 添加以下内容

```groovy
apply plugin: 'com.android.application' //放在这行下面
apply plugin: 'com.google.gms.google-services'
apply plugin: 'com.google.firebase.crashlytics'

```

## （国外）根目录的build.gradle增加以下内容

**注意下面注释的内容**

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
        //   当前使用->建议升级版本<br>
        //   3.3.x->3.3.3
        //   3.4.x->3.4.3
        //   3.5.x->3.5.4
        //   3.6.x->3.6.4
        //   4.0.x->4.0.1
        classpath 'com.android.tools.build:gradle:3.5.4'
 

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

## （国内）根目录的build.gradle增加以下内容

```groovy
buildscript {

    repositories {
        maven { url 'https://dl.bintray.com/umsdk/release' }
    }

    dependencies {

        //   当前使用->建议升级版本<br>
        //   3.3.x->3.3.3
        //   3.4.x->3.4.3
        //   3.5.x->3.5.4
        //   3.6.x->3.6.4
        //   4.0.x->4.0.1
        classpath 'com.android.tools.build:gradle:3.5.4'
        
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

## 引入广告依赖

在导入的库工程或者你的phone module的build.gradle 添加以下内容，注意区分国内国外

```groovy

repositories {
    flatDir {
        dirs 'libs'
    }
}
dependencies {
    
    //-----------------国内需要引入的--------------------

    //sdk核心库（国内必须）
    implementation 'com.eyu.opensdk:core-ch:1.7.33'

        //国内通常使用穿山甲
    implementation 'com.eyu.opensdk.ad.mediation:pangle-ch-adapter:3.4.1.2.31'
    
    //-----------------国外需要引入的--------------------

    //sdk核心库（国外必须）
    implementation 'com.eyu.opensdk:core:1.7.35'
    
    //国外通常使用max
    implementation 'com.eyu.opensdk.ad.mediation:max-adapter:10.1.1.30'

}
```

## 修改manifest文件

添加以下内容
```xml
<application
    android:fullBackupContent="@xml/custom_backup_rule"
    tools:replace="android:fullBackupContent"
>
```

# SDK使用

## 权限回调

如果是**国内**发布，在你的Activity中添加一下代码

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    SdkCompat.getInstance().onRequestPermissionsResult(this, requestCode, permissions, grantResults);
}
```

## 广告配置

广告配置有三个文件
> ad_setting.json
> ad_cache_setting.json
> ad_key_setting.json

实际给到的名字前面可能会有and_前缀，将它们放到引入的库工程中**res/raw**目录下<br>
确认下 ad_setting.json中的id字段和引入库工程中SDKHelper类下的字段一致

```java
public class SDKHelper {
    //激励视频 id
    private static final String REWARD_AD_PLACE_ID = "reward_ad";
    //插屏id
    private static final String INTERSTITIAL_AD_PLACE_ID = "inter_ad";
    //banner id
    private static final String BANNER_AD_PLACE_ID = "banner_ad";
}
```


## 其他配置

引入的库工程quick_start下的res/values/strings.xml是一个配置文件，将给到的配置填入即可

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <!-- 如果是国内，请改为false -->
    <bool name="is_overseas">true</bool>


    <!--facebook的app id，替换你自己的-->
    <string name="facebook_app_id"></string>
    <!--google广告的app id，替换你自己的-->
    <string name="google_ads_app_id"></string>
    <!--max广告sdk key，这个不用替换-->
    <string name="applovin_sdk_key">9aiR-8NXTyh84IIr1CWylMWBF4Fkf1Qpc8FaHc6qZoyd4FS6S-2BKZNaecIIdk6EtLq8zwzWiw8o9aucZxovU9</string>

    <!--appsflyer的key-->
    <string name="appsflyer_key"></string>

    <!--数数的key-->
    <string name="shushu_key"></string>

    <!--热云的key-->
    <string name="reyun_key"></string>

    <!--穿山甲的key和app name，一般国内接入需要-->
    <string name="pangle_key"></string>
    <string name="pangle_app_name"></string>

</resources>

```

## 广告使用示例

### 判断广告是否有可用的广告

```java
SDKHelper.isRewardAdLoaded()

SDKHelper.isInterAdLoaded()

SDKHelper.isBannerAdLoaded()
```

### 展示激励视频

```java
SDKHelper.showReward();
```

### 展示插屏

```java
SDKHelper.showInter();
```

### 展示banner

需要传入一个ViewGroup，这个group是用来放banner的

```java
SDKHelper.showBanner(ViewGroup);
```

# 测试

国外建议使用vpn切换到美国节点

# 事件埋点

## 基本数据埋点

调用下面的方法，事件会上传到umeng和Appsflyer，**不会上传到数数** 如果要上传到数数，请额外调用数数的方法
```java
//事件不带参数
EventHelper.getInstance().logEvent("事件名称");
//事件带map参数
EventHelper.getInstance().logEventWithParamsMap("事件名称",new HashMap<String, Object>());
//事件带json字符串参数
EventHelper.getInstance().logEventWithJsonParams("事件名称","json");
```

## 数数的埋点

**你可能会用到这个，看给你的配置中有没有数数的配置**
EventHelper对数数sdk的方法只是做了一层简单的封装，并没有做任何处理，所以先看下[数数的文档](https://docs.thinkingdata.cn/ta-manual/latest/installation/installation_menu/client_sdk/android_sdk_installation/android_sdk_installation.html#%E4%B8%89%E3%80%81%E5%8F%91%E9%80%81%E4%BA%8B%E4%BB%B6)，了解每个方法的含义


> EventHelper.getInstance().track("");

```java

EventHelper.getInstance().track("事件名称");

EventHelper.getInstance().track("事件名称",JSONObject);//参数

//更多方法
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

# 常见问题

+ sdk下载失败？  
  检查build.gradle配置是否添加，如果添加好后还是不能加载成功，请检查网络是否连通

+ 没有广告展示？  
  1.请检查广告配置是否正确配置，如果配置好了，在Android studio的日志打印那里过滤onAdLoadFailed，有错误码打印，将错误码提供给支持

+ 出现构建错误:unexpected element ＜queries＞ found in ＜manifest＞
  修改根目录build.gradle文件中的Android Gradle 插件版本
  当前使用->建议升级版本<br>
  3.3.x->3.3.3<br>
  3.4.x->3.4.3<br>
  3.5.x->3.5.4<br>
  3.6.x->3.6.4<br>
  4.0.x->4.0.1<br>

  举个例子，如果您正在使用 4.0.0 版本的 Android Gradle 插件，就可以在项目级别的 build.gradle 文件中将相关依赖升级到上图中对应的版本。

```groovy
buildscript {
    dependencies {

        //
        // classpath 'com.android.tools.build:gradle:4.0.0'
        classpath 'com.android.tools.build:gradle:4.0.1'
    }
}
```

