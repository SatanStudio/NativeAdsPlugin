![LOGO](https://api.pcloud.com/getpubthumb?code=XZuTNtZBs6bTEONAo5kt0Q1MpqlzkbL4m0k&linkpassword=undefined&size=144x144&crop=0&type=auto)

# NativeAdsPlugin v0.0.1

Cocos Creator 插件：实现原生平台广告的快速接入（支持iOS、Android）。

Creator编辑器进行广告信息配置，广告展示策略配置，编辑器下广告模拟展示，构建发布后快速集成至Xcode工程、Android Studio工程。

## 支持广告列表

| 名称 | 横幅  | 插屏 | 激励 |         说明         |
|:-----------------------:|:---------------------------:|:----------------------------:|:----------------------------:|:----------------------------:|
| Admob | y | y | n | https://www.google.com/admob/ |
| 广点通(腾讯广告) | y | y | n | http://e.qq.com/dev/index.html |
| Facebook | y | y | n | https://developers.facebook.com/docs/audience-network 仅支持海外用户 |
| Unity | n | y | y | https://unity3d.com/cn/services/ads |
| Vungle | n | y | y | http://vungle.cn/ |

## 插件安装方法

请参考 [扩展编辑器:安装与分享](http://www.cocos.com/docs/creator/extension/install-and-share.html) 文档。

CocosCreator Menu->插件->插件商店 购买下载安装后项目中多出native-ads-plugin-assets目录安装完成

## 运行DEMO

安装后，复制 creator工程目录/packages/native-ads-plugin/demo 至 creator工程目录/assets 下

creator编辑器下打开DEMO场景，[配置广告信息](#Edit_Config_Data)，构建、发布，[平台接入](#Platform_Specific)，enjoy！

## 快速开始

将命名NativeAdsManager的预制拖入场景中作为根节点。

### <a id="Edit_Config_Data"></a> 配置广告信息

在NativeAdsManager节点中NativeAdsConfigData组件配置广告信息与展示策略。

详解参数意义，以admob为例：

admobBannerID 横幅ID

admobIniterialID 插屏ID

admobBannerEnabled 是否启用admob横幅广告

admobEnabled 是否启用admob插屏和激励广告

admobPriority 插屏展示优先级数字越小优先级越高，目的尽量多的展示广告与最大化收益

admobCooldown 插屏、激励广告展示的冷却时间，目的防止频繁展示被广告平台封号

admobTestDevices 测试设备号，在xcode、android studio真机调试时日志中广告平台会打印出测试设备号添加在这，目的防止测试频繁展示被广告平台封号

unityEncouragePriority 激励展示优先级数字越小优先级越高，目的尽量多的展示广告与最大化收益

### 代码使用

#### 初始化广告管理类

初始化必须调用，推荐在程序开始。

```js
var adManager = cc.find('NativeAdsManager').getComponent('NativeAdsManager');
adManager.initialize();
```

初始化只有第一次有效，读取NativeAdsConfigData中的配置数据，如果希望动态配置广告信息等数据，需要在初始化前修改NativeAdsConfigData中的内容。
```js
var configData = cc.find('NativeAdsManager').getComponent('NativeAdsConfigData');
configData.admobBannerID = '123456';
configData.admobEnabled = true;
var adManager = cc.find('NativeAdsManager').getComponent('NativeAdsManager');
adManager.initialize();
```

#### 展示横幅广告

```js
adManager.showBanner();
```

#### 隐藏横幅广告

```js
adManager.hideBanner();
```

#### 展示插屏广告

```js
adManager.showInterstitial();
```

#### 查询是否有可展示的插屏广告

```js
bool flag = adManager.isInterstitialReady();
```

#### 展示激励广告

```js
adManager.showEncourageAd();

this.adManager.showEncourageAd('每日奖励');
this.adManager.showEncourageAd('复活');
```

#### 查询是否有可展示的激励广告

```js
bool flag = adManager.isEncourageAdReady();
```
#### 回调

```js
adManager.addListener(this, this.onAdsCallback);

onAdsCallback(event){
    cc.log('onAdsCallback eventName = ' + event.detail.eventName + ', adtype = ' + event.detail.adType);
    if (event.detail.eventName == 'encourageAdFinish'){
        cc.log('encourage zone = ' + event.detail.zone);
        //激励广告逻辑
        if (event.detail.zone == '每日奖励'){
            //每日奖励逻辑

        } else if (event.detail.zone == '复活'){
            //复活逻辑

        }
    }
},
```

## <a id="Platform_Specific"></a> 平台接入

在CocosCreator第一次构建发布后，还需要配置原生项目（仅第一次）

### iOS平台(Xcode工程)

#### 添加依赖库和OC代码到Xcode工程

Xcode目录 creator工程目录/build/jsb-default(不同模版名字不同如jsb-link、jsb-binary)/frameworks/runtime-src/proj.ios_mac

NativeAdsPlugin的iOS目录(要添加的依赖库和代码) creator工程目录/packages/native-ads-plugin/platforms/ios

打开Xcode工程，导入nativeAdsPlugin的iOS目录下的文件，勾选copy items if needed

编辑AppController.h添加:

```objc
@interface AppController : NSObject

...

@property (retain, nonatomic) RootViewController *viewController;

@end

extern "C"{
    
    AppController* getAppController ();
    
    UIViewController *getRootViewController();
}
```

编辑AppController.mm添加:

```objc
@implementation AppController

@synthesize viewController = viewController;

...

static AppController *_appController;

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    _appController = self;
    ...
}

...

@end


extern "C"{
    
    AppController* getAppController () {
        return _appController;
    }
    
    UIViewController *getRootViewController() {
        return getAppController().viewController;
    }
    
}

```

#### 添加Frameworks

AdSupport.framework

CoreTelephony.framework

libz.tbd

StoreKit.framework

libxml2.tbd

WebKit.framework

EventKitUI.framework

EventKit.framework

MessageUI.framework

![SS](https://api.pcloud.com/getpubthumb?code=XZ06AtZjJRjajvCPGhTt4zOP0dFrXGvyEey&linkpassword=undefined&size=636x355&crop=0&type=auto)


#### Compile Flags

新添加的.m和.mm文件需要标注自动引用计数-fobjc-arc

![SS](https://api.pcloud.com/getpubthumb?code=XZJ6AtZFf9VhxTQcmQVhjg6ThOMEQmjh3zk&linkpassword=undefined&size=640x532&crop=0&type=auto)


#### iOS10 新增权限验证

Info.plist添加以下权限

```plist
<key>NSBluetoothPeripheralUsageDescription</key>
<string>Use Your Bluetooth</string>
<key>NSCalendarsUsageDescription</key>
<string>Use Your Calendar</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>Use Your Location</string>
<key>NSLocationUsageDescription</key>
<string>Use Your Location</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Use Your Location</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Use Your Photo</string>
```

### Android平台(Android Studio工程)

#### 添加依赖库和java代码到Android Studio工程

Android Studio目录 creator工程目录/build/jsb-default/frameworks/runtime-src/proj.android-studio

NativeAdsPlugin的Android Studio目录 creator工程目录/packages/native-ads-plugin/platforms/android-studio

NativeAdsPlugin目录/src 文件复制到 Android Studio目录/app/src下

NativeAdsPlugin目录/libs 文件复制到 Android Studio目录/app/libs下

![SS](https://api.pcloud.com/getpubthumb?code=XZp6AtZ8wsyTkfuIBfBa4Cp4469SmIVEgdk&linkpassword=undefined&size=390x475&crop=0&type=auto)

#### 配置build.gradle文件
minSdkVersion 14+,compileSdkVersion 23+,targetSdkVersion=22
```gradle
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"
    
    defaultConfig {
        minSdkVersion 14
        targetSdkVersion 22
        ...
    }
    ...
    lintOptions {
        abortOnError false
    }
}


repositories {
    flatDir {
        dirs 'libs'
    }
    mavenCentral()
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile project(':libcocos2dx')

    compile 'com.google.firebase:firebase-ads:9.4.0'
    compile 'com.google.android.gms:play-services-gcm:9.4.0'
    compile 'com.alibaba:fastjson:1.2.21'
    compile 'com.facebook.android:audience-network-sdk:4.+'

    compile 'com.android.support:support-v4:23.4.0'
    compile 'com.android.support:appcompat-v7:23.4.0'

    compile files('libs/GDTUnionSDK.TBS.4.9.542.min.jar')
    compile(name: 'DebugSettings', ext: 'aar')
    compile(name: 'unity-ads', ext: 'aar')
}
```

#### 配置AndroidManifest.xml文件

添加权限

```xml
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

添加activity

```xml
<!--facebook-->
<activity android:name="com.facebook.ads.AudienceNetworkActivity"
    android:configChanges="keyboardHidden|orientation|screenSize" />
<!--gdt-->
<service
    android:name="com.qq.e.comm.DownloadService"
    android:exported="false"/>
<activity
    android:name="com.qq.e.ads.ADActivity"
    android:configChanges="keyboard|keyboardHidden|orientation|screenSize"/>
<!-- Vungle -->
<activity android:name="com.vungle.publisher.VideoFullScreenAdActivity"
    android:configChanges="keyboardHidden|orientation|screenSize|screenLayout|smallestScreenSize"
    android:theme="@android:style/Theme.NoTitleBar.Fullscreen"/>
<activity android:name="com.vungle.publisher.MraidFullScreenAdActivity"
    android:configChanges="keyboardHidden|orientation|screenSize|screenLayout|smallestScreenSize"
    android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen"/>
```

