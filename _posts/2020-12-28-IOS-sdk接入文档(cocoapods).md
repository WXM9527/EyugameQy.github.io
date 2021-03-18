---
title: IOS接入文档-cocoapods（推荐）
author: tangmingding
date: 2021-3-17 15:00:00 +0800
categories: [Blogging, Tutorial]
tags: [IOS]
pin: true
---

## Installation
本文档为通过cocoapods集成说明文档，若需手动集成SDK请跳转至[SDK接入文档手动集成](https://eyugameqy.github.io/posts/IOS-sdk接入文档(手动接入)/)

## 一.SDK集成
### 1、本SDK所有第三方sdk均可以模块形式集成，podfile的写法如下
```pod
pod 'EyuLibrary-ios',:subspecs => ['Core','模块一','模块二'], :git => 'https://github.com/EyugameQy/EyuLibrary-ios.git',:tag =>'1.3.91'
```

举例：
```pod
pod 'EyuLibrary-ios',:subspecs => ['Core','um_sdk', 'af_sdk', 'applovin_max_sdk','gdt_ads_sdk',  'firebase_sdk'], :git => 'https://github.com/EyugameQy/EyuLibrary-ios.git',:tag =>'1.3.91'
```

下面是所有模块及对应的需要添加的预编译宏
```pod
模块：易娱 sdk          :'Core'
    穿山甲（国内）       :'bytedance_ads_cn_sdk'       BYTE_DANCE_ADS_ENABLED
    穿山甲（国外）       :'bytedance_ads_global_sdk'   BYTE_DANCE_ADS_ENABLED
    广点通广告          :'gdt_ads_sdk'                GDT_ADS_ENABLED
    mtg广告            :'mtg_ads_sdk'                MTG_ADS_ENABLED
    FB广告             :'fb_ads_sdk'                 FACEBOOK_ENABLED FB_ADS_ENABLED
    unity广告          :'unity_ads_sdk'              UNITY_ADS_ENABLED
    Vungle广告         :'vungle_ads_sdk'             VUNGLE_ADS_ENABLED
    applovin广告       :'applovin_ads_sdk'           APPLOVIN_ADS_ENABLED
    ironsource广告     :'iron_ads_sdk'               IRON_ADS_ENABLED
    firebase          :'firebase_sdk'               FIREBASE_ENABLED
    AppsFlyer         :'af_sdk'                     AF_ENABLED
    广点通买量          :'gdt_action'                 GDT_ACTION_ENABLED
    FB登录             :'fb_login_sdk'               FACEBOOK_ENABLED FACEBOOK_LOGIN_ENABLED 
    热云               :'ReYunTracking'              TRACKING_ENABLED
    ADMOB             :'admob_sdk'                  ADMOB_ADS_ENABLED
    applovin MAX      :'applovin_max_sdk'           APPLOVIN_MAX_ENABLED
    TopOn(AnyThink)   :'anythink_sdk'               ANYTHINK_ENABLED
    AdmobMediation    :'admob_mediation_sdk'        ADMOB_MEDIATION_ENABLED ADMOB_ADS_ENABLED
    TradPlus          :'tradplus_sdk'               TRADPLUS_ENABLED
    穿山甲聚合(abu)     :'abu_ad_sdk'                 ABUADSDK_ENABLED
    sigmob            :'sigmob_ads_sdk'
    
    数数(Thinking)     :'thinking_sdk'               THINKING_ENABLED
    友盟               :'um_sdk'                     UM_ENABLED
    crashlytics       :'crashlytics_sdk'
    
注意：引入的模块的预编译宏在debug和release下均需添加
```

### 2、在终端运行 pod install

### 3、执行成功后，用xcode打开当前项目目录下的xcworkspace文件

## 二.SDK初始化
### SDK初始化流程
```oc
#import "EYAdManager.h"
#import "EYAdConfig.h"
#import "EYSdkUtils.h"

//某些平台通过EYSdkUtils初始化
[EYSdkUtils initFirebaseSdk];
[EYSdkUtils initUMMobSdk:@"XXXXXXXXXXXXXXXXXX" channel:@"channel"];
[EYSdkUtils initAppFlyer:@"XXXXXXXXXXXXXXXXX" appId:@"XXXXXXXXXXXXX"];
[EYSdkUtils initGDTActionSdk:@"XXXXXX" secretkey:@"XXXXXXXXX"];

//firebase 远程配置初始化: 引入EYRemoteConfigHelper.h头文件  
NSDictionary* dict = [[NSDictionary alloc] init];
[[EYRemoteConfigHelper sharedInstance] setupWithDefault:dict];

//读取配置文件
EYAdConfig* adConfig = [[EYAdConfig alloc] init];
adConfig.adKeyData =  [EYSdkUtils readFileWithName:@"ios_ad_key_setting"];
adConfig.adGroupData = [EYSdkUtils readFileWithName:@"ios_ad_cache_setting"];
adConfig.adPlaceData = [EYSdkUtils readFileWithName:@"ios_ad_setting"];

//某些平台通过设置adConfig初始化
adConfig.unityClientId = @"XXXXXX";
adConfig.vungleClientId = @"XXXXXXXXXXXXXXX";
adConfig.ironSourceAppKey = @"XXXXXXX";
adConfig.wmAppKey = @"XXXXXX";

//如果有banner广告需要设置根控制器
[[EYAdManager sharedInstance] setRootViewController:window.rootViewController];

//如果集成了AdmobMediation且配置了vungle广告
[EYAdManager sharedInstance].vunglePlacementIds = @[placepentId1, placepentId2...];

[[EYAdManager sharedInstance] setupWithConfig:adConfig];
```
各平台初始化详细配置请阅读以下内容

### 1、applovin MAX
```txt
max  需要在GCC_PREPROCESSOR_DEFINITIONS 加上 APPLOVIN_MAX_ENABLED
在info.plist里设置AppLovinSdkKey以及FacebookAppID
```

### 2、穿山甲SDK 
```txt
在GCC_PREPROCESSOR_DEFINITIONS 加上 BYTE_DANCE_ADS_ENABLED
adConfig.wmAppKey = @"XXXXXX";//代码里设置穿山甲sdk app key
```

### 3、广点通广告
```txt
广点通广告 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 GDT_ADS_ENABLED
adConfig.gdtAppId = @"xxxxxxxxxx";//代码里设置广点通广告sdk app id
```

### 4、mtg广告
```txt
mtg广告 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 MTG_ADS_ENABLED
adConfig.mtgAppId = @"xxxxxx";//代码里设置mtg广告sdk app id 及app key
adConfig.mtgAppKey = @"xxxxxxxxxxxxxxxxx";
```

### 5、FB广告
FB广告 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 FB_ADS_ENABLED 及FACEBOOK_ENABLED   
请参考https://developers.facebook.com/docs/app-events/getting-started-app-events-ios   
在app 对应的生命周期函数里加上   
```oc
[EYSdkUtils initFacebookSdkWithApplication:application options:launchOptions];
[EYSdkUtils application:app openURL:url options:options];
```
在Info.plist文件里加上
```xml
    <key>FacebookAppID</key>
    <string>xxxxxx</string>
    <key>FacebookDisplayName</key>
    <string>xxxxxx</string>
```
### 6、unity广告
```txt
unity广告 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 UNITY_ADS_ENABLED
adConfig.unityClientId = @"xxxxxxx";
```
### 7、Vungle广告
```txt
Vungle广告 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 VUNGLE_ADS_ENABLED
adConfig.vungleClientId = @"xxxxxxxxxxx";
```

### 8、applovin广告
```txt
applovin广告 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 APPLOVIN_ADS_ENABLED
AppLovin需要在info.plist里设置AppLovinSdkKey
```

### 9、ironsource广告
```txt
ironsource广告 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 IRON_ADS_ENABLED
adConfig.ironSourceAppKey = @"xxxxxxxx";
```

### 10、firebase 及 crashlytics 以及ADMOB
#### firebase
```txt
参考资料：https://firebase.google.com/docs/ios/setup?authuser=0
从firebase下载 GoogleService-Info.plist 并放到xcode的根目录
在GCC_PREPROCESSOR_DEFINITIONS 加上 FIREBASE_ENABLED 
[EYSdkUtils initFirebaseSdk];
```

#### admob
```txt
admob可以通过"firebase_sdk"模块或者"admob_sdk"模块来引入，海外使用firebase_sdk，国内使用admob_sdk
在GCC_PREPROCESSOR_DEFINITIONS 加上 ADMOB_ADS_ENABLED
```
Info.plist 加上以下内容
```xml
<key>GADIsAdManagerApp</key>
<true/>
```
配置admob client id
```oc
adConfig.admobClientId = @"ca-app-pub-7585239226773233~4631740346";
```

### 11、友盟
```txt
需要在GCC_PREPROCESSOR_DEFINITIONS 加上 UM_ENABLED
[EYSdkUtils initUMMobSdk:@"XXXXXXXXXXXXXXXXXX" channel:@"channel"];
```

### 12、AppsFlyer
```txt
需要在GCC_PREPROCESSOR_DEFINITIONS 加上 AF_ENABLED
[EYSdkUtils initAppFlyer:@"XXXXXXXXXXXXXXXXX" appId:@"XXXXXXXXXXXXX"];
```

### 13、广点通买量
```txt
需要在GCC_PREPROCESSOR_DEFINITIONS 加上 GDT_ACTION_ENABLED
[EYSdkUtils initGDTActionSdk:@"XXXXXX" secretkey:@"XXXXXXXXX"];
并在- (void)applicationDidBecomeActive:(UIApplication *)application中调用[EYSdkUtils doGDTSDKActionStartApp];
```

### 14、FB登录
```txt
需要在GCC_PREPROCESSOR_DEFINITIONS 加上 FACEBOOK_LOGIN_ENABLED
```

### 15、热云
```txt
热云  需要在GCC_PREPROCESSOR_DEFINITIONS 加上 TRACKING_ENABLED
[EYSdkUtils initTrackingWithAppKey:appKey];
```

### 16、TopOn(AnyThink)
```txt
AnyThink  需要在GCC_PREPROCESSOR_DEFINITIONS 加上 ANYTHINK_ENABLED
并集成需要用到的广告模块，比如用到了admob广告则需要额外添加"admob_sdk"模块，或者自己额外集成admob对应版本的的SDK
[EYSdkUtils initAnyThinkWithAppID:appid AppKey:appKey];
```

### 17、AdmobMediation
```txt
AdmobMediation  需要在GCC_PREPROCESSOR_DEFINITIONS 加上 ADMOB_MEDIATION_ENABLED ADMOB_ADS_ENABLED
配置并初始化admob
fb广告需要在info.plist里设置FacebookAppID
如果有vungle广告需要配置 [EYAdManager sharedInstance].vunglePlacementIds = [placepentId1, placepentId2...];
```

### 18、Thinking（数数）
```txt
Thinking  需要在GCC_PREPROCESSOR_DEFINITIONS 加上 THINKING_ENABLED 

//默认使用云服务，服务器地址: https://receiver.ta.thinkingdata.cn 
[EYSdkUtils initThinkWithAppID:APP_ID]; 

//如果您使用的是私有化部署的版本请传入自己的url地址 
[EYSdkUtils initThinkWithAppID:APP_ID Url:SERVER_URL]; 
```

### 19、TradPlus
```txt
TradPlus  需要在GCC_PREPROCESSOR_DEFINITIONS 加上 TRADPLUS_ENABLED 
Admob要求  Info.plist 添加 GADApplicationIdentifier
fb广告需要在info.plist里设置FacebookAppID
```

### 19、穿山甲聚合（abu）
```txt
ABUAd 需要在GCC_PREPROCESSOR_DEFINITIONS 加上 ABUADSDK_ENABLED 
adConfig.abuAppId = APP_ID;
```

## 三.加载广告
一般来说cache配置文件默认配置isAutoLoad的值为true，表示SDK初始化后将自动加载缓存广告,可直接跳过此步骤,若需手动加载则将对应的值改为false，并手动调用load方法加载广告
```oc
//加载插屏广告
[[EYAdManager sharedInstance] loadInterstitialAd: placeId];

//加载激励视频
[[EYAdManager sharedInstance] loadRewardVideoAd: placeId];

//加载原生广告
[[EYAdManager sharedInstance] loadNativeAd: placeId];

//加载Banner
[[EYAdManager sharedInstance] loadBannerAd: placeId];

//加载开屏广告
[[EYAdManager sharedInstance] loadSplashAd: placeId];

//判断广告是否加载完成
bool isNativeAdLoaded = [[EYAdManager sharedInstance] isNativeAdLoaded: placeId];
bool isBannerAdLoaded = [[EYAdManager sharedInstance] isBannerAdLoaded: placeId];
bool isInterstitialAdLoaded = [[EYAdManager sharedInstance] isInterstitialAdLoaded: placeId];
bool isRewardAdLoaded = [[EYAdManager sharedInstance] isRewardAdLoaded: placeId];
bool isSplashAdLoaded = [[EYAdManager sharedInstance] isSplashAdLoaded: placeId];
```

## 四.显示广告
```oc
//展示激励视频 reward_ad为广告位id，对应ios_ad_setting.json配置
[[EYAdManager sharedInstance] showRewardVideoAd:@"reward_ad" withViewController:self];

//展示插屏 inter_ad为广告位id，对应ios_ad_setting.json配置
[[EYAdManager sharedInstance] showInterstitialAd:@"inter_ad" withViewController:self];

//展示原生广告 native_ad为广告位id，对应ios_ad_setting.json配置
[[EYAdManager sharedInstance] showNativeAd:@"native_ad" withViewController:self viewGroup:self.nativeRootView];

//展示Banner广告 banner_ad为广告位id，对应ios_ad_setting.json配置
bool isSuccess = [[EYAdManager sharedInstance] showBannerAd:@"banner_ad" viewGroup:self.bannerRootView];

//展示Splash广告 splash_ad为广告位id，对应ios_ad_setting.json配置
[[EYAdManager sharedInstance] showSplashAd:@"splash_ad" withViewController:self];
```

