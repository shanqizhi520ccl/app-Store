# 神起游戏SDK-AppStore版-开发参考说明书(iOS API)
######  建议在游戏资源载入后接入SDK相关步骤（初始化-->登录-->进入游戏（同步角色信息）-->...）

**修订记录** 

| **归档日期** | **版本** | **说明** | **作者** | **审批人** |
| ------------ | -------- | -------- | -------- | ---------- |
| 2021–09-08 | 1.0000  |  初始版本(初稿)  |  单启志  |  <font color='red'>---</font>  |
| 2021–10-20 | 1.0001  |  SDK加密方案优化  |  单启志  |  ---  |
| - |  - |  -  |  -  |  -  |

## 1.使用说明 

### 1.1 集成说明 

在接入时，仅需关注文档中所提供的接口，而不用了解具体实现。

### 1.2 系统支持 

目前SDK支持iOS9.0及以上版本。


## 2. 使用准备 

### 2.1 下载SDK包和Demo 

```
下载SDK及文档，得到相关文件。压缩包内包括：
1. SDKDemo
2. Sdks（包含:BXMobileSDK.bundle+BXMobileSDK.framework）
3. Document
```

### 2.2 编译选项<font color='red'>(参考)</font>

1. 修改Other Linker Flags：SDK内部使用了Objective-C的Category，所以开发者需要在工程的 `Targets -> Build Settings -> Linking –> Other Linker Flags` 中添加`–ObjC`选项，以保证这些Category能够正常使用。

<font color='red'>注意区分-ObjC 的大小写，以防 Crash</font>

### 2.3 添加配置<font color='red'>(必接)</font>

1. **在使用苹果内购时，需在 Xcode 工程的 `Targets -> Capabilities` -> 打开 `In-App Purchase`

2. **由于 SDK 内部加密协议涉及 KeyChain，如使用 Xcode8 及以上环境开发，在运行调试时请开启 `Targets -> Signing & Capabilities -> Capability -> KeyChain Sharing` 选项


### 2.4 横竖屏配置<font color='red'>(可选)</font>

1. **竖屏： 在工程的 `Targets -> General` 下勾选竖屏：`Portrait`

2. **横屏： 在工程的  `Targets -> General` 下勾选左右横屏：`Landscape Left` + `Landscape Right`



## 3. 接口与使用说明<font color='red'>(步骤说明：初始化-->登录-->进入游戏（同步角色信息）-->...)</font>

### 3.1 基础配置<font color='red'>(必接)</font>

在info.plist中，添加/修改如下配置: 

```xml
<key>NSAppTransportSecurity</key>
	<dict>
		<key>NSAllowsArbitraryLoads</key>
		<true/>
	</dict>
    
<key>NSCameraUsageDescription</key>
<string>需要使用您的相机照相</string>
<key>NSPhotoLibraryAddUsageDescription</key>
<string>需要保存图片到您的图库</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>需要使用您的图库</string>

<key>CFBundleDevelopmentRegion</key>
<string>zh_CN</string>

```

### 3.2 渠道选择<font color='red'>(注意)</font>

**在 `BXMobileSDK.bundle` 包的 `PlatformConfig.plist` 配置**

1. **其中 `platformId` 为渠道ID，参考物料信息**

2. **其中 `isTestFlight` 为Bool类型，审核服：<font color='red'>0</font>  与  正式服：<font color='red'>1</font>**

3. **其中 `reference` 为渠道ID对应的渠道名称，不需要变化**


### 3.3 添加SDK回调通知<font color='red'>(必接)</font>

```objective-c 
    // 监听初始化
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(registerAppSuccessNotification:)
                                                 name:BXRegisterAppSuccessNotification
                                               object:nil];
    
    // 监听登录
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(loginSuccessNotification:)
                                                 name:BXUserLoginSuccessNotification
                                               object:nil];
    
    // 监听注销
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(logoutAppSuccessNotification:)
                                                 name:BXLogoutAppSuccessNotification
                                               object:nil];
                                               

    // 初始化成功通知
    - (void)registerAppSuccessNotification:(NSNotification *)notification {
        NSLog(@"--初始化完成回调-- %@", notification.object);
        //注意: 登录注册等其他操作必须是在初始化成功后调用
        if ([BXMobileManager shareManager].appServer) {
        // 游戏正式服处理逻辑
        }else{
        // 游戏审核服处理逻辑
        }
    }

    // 登录成功通知
    - (void)loginSuccessNotification:(NSNotification *)notification {
        NSDictionary *result = (NSDictionary *)notification.object;
        NSLog(@"登录成功 result = %@", result);
        // your own code .....
    
        /// 开启悬浮窗（可选）
        [BXMobileManager bx_showButtonWindow];
    
    }

    // 登出通知
    - (void)logoutAppSuccessNotification:(NSNotification *)notification {
        NSLog(@"--登出完成回调-- %@", notification.object);
    
        /// 隐藏悬浮窗（可选）如遇混淆打乱：bx_showButtonWindow下面的方法便是
        [BXMobileManager bx_dissmissButtonWindow];
    }
```

### 3.3 初始化SDK<font color='red'>(必接，二选一)</font>

```objective-c
        ///初始化（普通）
		/**
		 * SDK initialize.
		 * @param appId 渠道分配 appId
		 * @param appKey 渠道分配 appKey
		 */
        [BXMobileManager registerAppId:@"X36" appKey:@"bac1da6c87ee026bea64f059680ac95a"]
        
        /**
 		* SDK series initialize.
		 * @param appId 渠道分配 appId
 		* @param seriesId 系列id seriesId
 		* @param appKey 渠道分配 appKey
		*/
        //    [BXMobileManager registerSeriesAppId:@"X36" seriesId:@"171" appKey:@"33f2baf3abe2789705881575c5e29f42"];
```
|      字段      |       说明       |
| :------------: | :--------------: |
|     appId      | 平台的appID  |
|     appKey     | 平台的appKey |
|     seriesId     | 平台的系列appID |

### 3.4 登录<font color='red'>(必接)</font>

登录接口必须在初始化后的回调方法中实现

```objective-c
	[[BXMobileManager shareManager] login];

	***登录回调走通知回调***
```

### 3.4 进入/升级/退出游戏(游戏角色信息相关)<font color='red'>(必接)</font>

```objective-c
    NSDictionary * userInfo = @{
        BXUserInfoRoleServerIdKey: self.zoneid.text,
        BXUserInfoRoleServerNameKey: self.zonename.text,
        BXUserInfoRoleIdKey: self.roleid.text,
        BXUserInfoRoleNameKey: self.rolename.text,
        BXUserInfoRoleLevelKey: self.level.text,
        BXUserInfoRoleGameCoinKey: @(self.coin.text.intValue),
    };
    
        //进入游戏（同步角色信息）
            [[BXMobileManager shareManager] updateUserRoleInfo:userInfo reportType:BXUserInfoReportType_EnterGame];
            break;
        
        //角色升级
            [[BXMobileManager shareManager] updateUserRoleInfo:userInfo reportType:BXUserInfoReportType_LevelUp];
            break;
            
        //退出游戏
            [[BXMobileManager shareManager] updateUserRoleInfo:userInfo reportType:BXUserInfoReportType_LogoutGame];
            break;
```

### 3.5 注销（退出账号）<font color='red'>(可选，回调必接 )</font>

```objective-c
	// 注销（退出账号）
   [[BXMobileManager shareManager] logout];
   
   **3.7.1退出账号回调走通知回调(必接)**
```

### 3.6 订单支付<font color='red'>(必接)</font>

代码示例： 

```objective-c
        NSDictionary *orderInfo = @{
        BXOrderInfoProductNameKey:@"60灵玉",//产品描述
        BXOrderInfoProductDescriptionKey:@"购买后，获得60灵玉", //产品描述
        BXOrderInfoOrderIDKey:[self randomOrder], //CP订单号
        BXOrderInfoExternKey:@"extension|data|test",//游戏自定义数据，充值成功，回调游戏服的时候，会原封不动返回,
        BXOrderInfoNotifyUrlKey:@"http://www.game.com/pay/callback",//支付成功，Server异步通知该地址，告诉游戏服务器发货
        //内购ID和内购价格
        BXOrderInfoProductIdKey:@"com.hysj.xy.shenyinggd.1",//内购ID
        BXOrderInfoProductPriceKey:@600,//6元(单位分), 必须是内购商品对应的金额

        //第三方支付时，CP后台定义的产品id和产品价格，非必须，如果不传则默认取内购ID和内购价格
        BXOrderInfoExternProductIdKey:@"id_from_server",//CP后台定义的产品id
        BXOrderInfoExternProductPriceKey:@600,//6元(单位分)
    };

        [[BXMobileManager shareManager] makeStore:orderInfo];
    
    ***订单最终支付状态服务端获取***
```
