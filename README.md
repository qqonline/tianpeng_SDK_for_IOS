# 天鹏广告IOS SDK接入文档

## 1 概述

尊敬的开发者朋友，欢迎您使用天鹏广告sdk平台。通过本文档，您可以轻松的在几分钟之内完成广告的集成过程。</br>
开发工具： Xcode 8 及以上版本</br>
操作系统： iOS 8.0 及以上版本</br>

## 2 Demo以及sdk下载
  
  [链接](https://gitlab.tianpengnet.cn/client/ios/tpadsdk)


## 3 SDK接入流程
### 3.1 导入SDK

### 必须导入的
* `TPCore.framework` 和 `TPCore.bundle`
* `TPNetworking.framework`
* `TPUtilsKit.framework`

### 可选广告平台 (资源位于文件夹中)
具体需要哪些平台, 请找商务确认
注意: 每个平台中的 .bundle, .framework, .a 缺一不可 
* `广点通` 
* `百度`
* `头条`
* `有道`


### 3.2 环境配置

1. 依赖库添加
```obj-c
Photos.framework
AdSupport.framework 
CoreLocation.framework 
QuartzCore.framework 
SystemConfiguration.framework
CoreTelephony.framework
libz.tbd 
WebKit.framework (Optional)
libxml2.tbd
Security.framework 
StoreKit.framework
MessageUI.framework
libc++.dylib
CoreGraphics.framework
EventKit.framework
EventKitUI.framework
SafariServices.framework
CoreMotion.framework
MediaPlayer.framework
MobileCoreServices.framework
CoreMedia.framework
AVFoundation.framework
libsqlite3.dylib
libresolv.9.tbd
```

2. 打开项目的 app target，查看 Build Settings 中的 Linking-Other Linker Flags 选项，确保含有 -ObjC 一值， 若没有则添加。

3. 在项目的 app target 中，查看 Build Settings 中的 Build options - Enable Bitcode 选项， 设置为NO。 

4. info.plist 添加支持 Http访问字段
```obj-c
<key>NSAppTransportSecurity</key>
<dict>
<key>NSAllowsArbitraryLoads</key>
<true/>
</dict>
```

5. Info.plist 添加定位权限字段
```obj-c
NSLocationWhenInUseUsageDescription
NSLocationAlwaysAndWhenInUseUsageDeion
```
## 4 接入代码
### 4.1 SDK初始化
1. 初始化方法
```obj-c
#import <TPCore.h>

TPConfiguration *configure = [[TPConfiguration alloc] initWithAppId:@"APP_ID"];
[[TPCore sharedInstance] initializeSDKWithConfiguration:configure completion:nil];
```

2. 如果接入了有道, 请根据有道后台的广告位配置, 配置对应的样式名称
```obj-c
#import <TPCore.h>

TPConfiguration *configure = [[TPConfiguration alloc] initWithAppId:@"APP_ID"];
[[TPCore sharedInstance] initializeSDKWithConfiguration:configure completion:nil];
```

2. 开启定位服务, 更加精准的投放广告
```obj-c
[TPCoreKit setLocationOn:YES];
```

### [注意]: 如果请求不到广告, 请通过以下API确认广告平台是否导入
```obj-c
// 查看工程中所有导入的平台 
[TPCoreKit allMediatedPlatformForApplication]

// 查看工程中已经注册有效的平台 (可以拉取广告)
[TPCoreKit registedMediatedPlatforms]
```
### 4.2 开屏广告

1. 请求示例
```obj-c
#import <TPSplashAd.h>

// 1. 初始化
self.splashAd = [[TPSplashAd alloc] init];
// 2. 设置默认启动图
self.splashAd.backgroundImage = [UIImage imageNamed:@"5cb1b947df8a5e38883f177c-banner.jpg"];
// 3. 设置代理回调
self.splashAd.delegate = self;
// 4. 设置弹出控制器
self.splashAd.controller = self;

// 5. 自定义底部View，可以在此View中设置应用Logo, 注意:bottomView需设置好宽高，所占的空间不能过大，并保证高度不超过屏幕高度的 25%
UIView *bottomView = [UIView new];
bottomView.backgroundColor = [UIColor redColor];
bottomView.frame = CGRectMake(0, 0, kTPScreenWidth, kTPScreenHeight * 0.25);
// 6. 展示
[self.splashAd loadAndShowWithBottomView:bottomView];
```

### 4.3 横幅广告

### banner高度不能低于50, 一般宽高比为320 * 50

1. 请求示例
```obj-c
#import <TPBannerAdView.h>

if (!self.bannerView) {
    // 1 初始化, 并设置宽高
    self.bannerView = [[TPBannerAdView alloc] initWithFrame:CGRectMake(0, 200, 320, 50)];
    // 2 设置点击弹出控制器
    self.bannerView.controller = self;
    // 3 设置生命周期代理
    self.bannerView.delegate = self;

    [self.view addSubview:self.bannerView];
    }
// 加载数据
[self.bannerView loadAndShow];
```

### 4.4 插屏广告

1. 初始化并预加载数据
```obj-c
#import <TPInterstitialAd.h>

self.interstitialAd = [[TPInterstitialAd alloc] init];
self.interstitialAd.delegate = self;
[self.interstitialAd loadAdData];
```

2. 弹出插屏广告
```obj-c
- (void)tp_interstitialAdDidLoad:(TPInterstitialAd *)interstitialAd {
    [interstitialAd presentFromRootViewController:self];
}
```

### 4.5 信息流广告

1. 初始化 (一个请求对象可以多次使用, 所以请采用懒加载)
```obj-c
#import "TPNativeAd.h"

if (_nativeAd) {
    /**
    信息流请求对象的构造方法

    @param expectAdSize 期望的广告位尺寸, 注意: 宽度必须精确, SDK内部会对高度进行自适应
    @param style 广告的展示样式, 具体请查看 'TPNativeAdStyle' 介绍
    @return 请求对象
    */
    _nativeAd = [[TPNativeAd alloc] initWithExpectAdSize:CGSizeMake(self.view.frame.size.width, 100) withStyle:TPNativeAdStylePicUp];
    _nativeAd.controller = self;
    _nativeAd.delegate = self;
}
```

2. 拉取信息流视图
```obj-c
/**
广告发起请求方法
详解：[必选]发起拉取广告请求,在获得广告数据后回调delegate

@param count 一次拉取广告的个数, 建议区间 1~5, 超过可能无法获取到, 或者获取时间太长
*/
[_nativeAd loadAd:1];
```

3. 在数据获取成功回调中, 渲染模板视图
```obj-c
/**
请求信息流广告成功

@param nativeAd 广告请求对象
@param adViews 信息流广告视图
*/
- (void)tp_nativeAdRequestSuccess:(TPNativeAd *)nativeAd adViews:(NSArray<__kindof TPNativeAdView *> *)adViews {
    for (TPNativeAdView *adView in adViews) {
        [adView render];
    }
}
```

4. 视图渲染成功回调中, 刷新页面
```obj-c
/**
信息流广告视图渲染成功

@param adView 广告视图
*/
- (void)tp_nativeAdViewRenderSuccess:(TPNativeAdView *)adView {
    [self.adviewItems addObject:adView];
    [self.tableView reloadData];
}
```

## 5. 常见问题

 广告未放量，请联系商务</br>

 广告渠道未接入，请联系商务

## 6. 版权

Copyright 2019 杭州天鹏网络技术有限公司.  </br> 

## 7. 联系方式

客服热线：136 3418 8697  </br>

邮箱： qiuchanghong@tianpengnet.com </br>

QQ: 286043810@qq.com </br>
 
