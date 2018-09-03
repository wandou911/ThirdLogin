# ThirdLogin
微信微博QQ第三方登陆

### 1 微信登陆


### 2 [微博登陆](http:// open.weibo.com) [微博iossdk-github地址](https://github.com/sinaweibosdk/weibo_ios_sdk)

[详细文档参考官方指南]()
1.注册成为开发者，创建移动应用

2.设定授权回调页
3.设定Apple ID 和 Bundle ID
4.设置工程回调URL Scheme
5.添加SDK文件到工程

快速集成 WeiboSDK支持使用Cocoapods集成，请在Podfile中添加以下语句
```
pod "Weibo_SDK", :git => "https://github.com/sinaweibosdk/weibo_ios_sdk.git" 
```
6.在工程中引入静态库之后,需要在编译时添加 –objC 编译选项
  
7.添加FrameWork文件到工程

8.定义应用 SSO 登录或者 Oauth2.0 认证所需的几个常量

```
#define kAppKey         @"4059482444"
#define kRedirectURI    @"https://api.weibo.com/oauth2/default.html"
```
9.注册 appkey(clientid) 

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [WeiboSDK enableDebugMode:YES];
    [WeiboSDK registerApp:kAppKey];
    return YES;
}
```
10.重写AppDelegate 的handleOpenURL和openURL方法
```
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    return [WeiboSDK handleOpenURL:url delegate:self];
}

- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
    return [WeiboSDK handleOpenURL:url delegate:self ];
}
```
### 3 QQ登陆


