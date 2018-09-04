# ThirdLogin
微信微博QQ第三方登陆

### 1 微信登陆


### 2 ![微博登陆](http:// open.weibo.com) [微博ios-sdk-github地址](https://github.com/sinaweibosdk/weibo_ios_sdk)

[详细文档参考官方指南]()

1.注册成为开发者，创建移动应用
  ![注册](https://ws1.sinaimg.cn/large/006tNbRwly1fuwhj3n0jyj31580e8ave.jpg)
  
  ![ios](https://ws4.sinaimg.cn/large/006tNbRwly1fuwhldqtnzj30rg05utcm.jpg)
  
2.设定授权回调页
  ![授权](https://ws4.sinaimg.cn/large/006tNbRwly1fuwhlq29t2j31540nwk1k.jpg)
3.设定Apple ID 和 Bundle ID
  ![appid](https://ws4.sinaimg.cn/large/006tNbRwly1fuwhmhc2jcj315g0lyqaq.jpg)
  
  ![bundle id](https://ws4.sinaimg.cn/large/006tNbRwly1fuwhn09bdhj315m0aun1z.jpg)
  
  
4.设置工程回调URL Scheme
  ![url scheme](https://ws1.sinaimg.cn/large/006tNbRwly1fuwhngwdk8j313u0boae0.jpg)
  
5.添加SDK文件到工程

快速集成 WeiboSDK支持使用Cocoapods集成，请在Podfile中添加以下语句
```
pod "Weibo_SDK", :git => "https://github.com/sinaweibosdk/weibo_ios_sdk.git" 
```
6.在工程中引入静态库之后,需要在编译时添加 –objC 编译选项
  
7.添加FrameWork文件到工程

8.定义应用 SSO 登录或者 Oauth2.0 认证所需的几个常量

```
#define kAppKey         @"2045436852"
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

11.在AppDelegate 中实现WeiboSDKDelegate的协议方法：

```
- (void)didReceiveWeiboRequest:(WBBaseRequest *)request
- (void)didReceiveWeiboResponse:(WBBaseResponse *)response
```
方法的具体实现：

AppDelegate.h
```
@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow *window;

@property (strong, nonatomic) NSString *wbtoken;
@property (strong, nonatomic) NSString *wbRefreshToken;
@property (strong, nonatomic) NSString *wbCurrentUserID;

@end

```

AppDelegate.m

```
- (void)didReceiveWeiboRequest:(WBBaseRequest *)request
{
    
}

- (void)didReceiveWeiboResponse:(WBBaseResponse *)response
{
    if ([response isKindOfClass:WBSendMessageToWeiboResponse.class])
    {
        NSString *title = NSLocalizedString(@"发送结果", nil);
        NSString *message = [NSString stringWithFormat:@"%@: %d\n%@: %@\n%@: %@", NSLocalizedString(@"响应状态", nil), (int)response.statusCode, NSLocalizedString(@"响应UserInfo数据", nil), response.userInfo, NSLocalizedString(@"原请求UserInfo数据", nil),response.requestUserInfo];
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title
                                                        message:message
                                                       delegate:nil
                                              cancelButtonTitle:NSLocalizedString(@"确定", nil)
                                              otherButtonTitles:nil];
        WBSendMessageToWeiboResponse* sendMessageToWeiboResponse = (WBSendMessageToWeiboResponse*)response;
        NSString* accessToken = [sendMessageToWeiboResponse.authResponse accessToken];
        if (accessToken)
        {
            self.wbtoken = accessToken;
        }
        NSString* userID = [sendMessageToWeiboResponse.authResponse userID];
        if (userID) {
            self.wbCurrentUserID = userID;
        }
        [alert show];
    }
    else if ([response isKindOfClass:WBAuthorizeResponse.class])
    {
        NSString *title = NSLocalizedString(@"认证结果", nil);
        NSString *message = [NSString stringWithFormat:@"%@: %d\nresponse.userId: %@\nresponse.accessToken: %@\n%@: %@\n%@: %@", NSLocalizedString(@"响应状态", nil), (int)response.statusCode,[(WBAuthorizeResponse *)response userID], [(WBAuthorizeResponse *)response accessToken],  NSLocalizedString(@"响应UserInfo数据", nil), response.userInfo, NSLocalizedString(@"原请求UserInfo数据", nil), response.requestUserInfo];
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title
                                                        message:message
                                                       delegate:nil
                                              cancelButtonTitle:NSLocalizedString(@"确定", nil)
                                              otherButtonTitles:nil];
        
        self.wbtoken = [(WBAuthorizeResponse *)response accessToken];
        self.wbCurrentUserID = [(WBAuthorizeResponse *)response userID];
        self.wbRefreshToken = [(WBAuthorizeResponse *)response refreshToken];
        [alert show];
    }
}
```

问题：1
-canOpenURL: failed for URL: “sinaweibo” - error:”This app is not allowed to query for scheme xx 
-canOpenURL: failed for URL: “weibosdk” - error:”This app is not allowed to query for scheme xx

原因：缺少白名单字段

解决方式：在plist中的LSApplicationQueriesSchemes添加字段 

![白名单](https://ws2.sinaimg.cn/large/006tNbRwly1fux94mjsmwj30h30aq75r.jpg)

### 3 QQ登陆


