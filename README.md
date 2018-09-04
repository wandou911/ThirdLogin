# ThirdLogin
微信微博QQ第三方登陆

### 1 微信登陆


### 2 微博登陆(http://open.weibo.com)  [微博ios-sdk-github地址](https://github.com/sinaweibosdk/weibo_ios_sdk)

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

![–objC编译选项](https://ws2.sinaimg.cn/large/006tNbRwly1fuxchgrosfj315a0bqdn3.jpg)
  
7.添加FrameWork文件到工程

![添加FrameWork](https://ws3.sinaimg.cn/large/006tNbRwly1fuxciml3zej31520g4wlf.jpg)

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
```
<key>LSApplicationQueriesSchemes</key>
	<array>
		<string>sinaweibo</string>
		<string>weibosdk</string>
		<string>weibosdk2.5</string>
		<string>sinaweibohd</string>
	</array>
	<key>LSRequiresIPhoneOS</key>
```

![白名单](https://ws2.sinaimg.cn/large/006tNbRwly1fux94mjsmwj30h30aq75r.jpg)

### 3 QQ登陆

#### 一 登陆QQ互联申请appid(https://connect.qq.com)
1.登陆qq互联

![QQ互联](https://ws1.sinaimg.cn/large/006tNbRwly1fuxd22ugx8j31kw0wxn49.jpg)

2.创建移动应用

![创建应用](https://ws3.sinaimg.cn/large/006tNbRwly1fuxilmg8vij31kw0x6jx9.jpg)

3.填入应用信息

![应用信息](https://ws3.sinaimg.cn/large/006tNbRwly1fuxim0tk6oj31kw11bn3i.jpg)

4.完善应用信息

![完善](https://ws2.sinaimg.cn/large/006tNbRwly1fuxir8pp0yj31kw13mtej.jpg)

5.查看应用appid

![appid](https://ws3.sinaimg.cn/large/006tNbRwly1fuximct24qj31js0ncjwi.jpg)


#### 二 集成

1. 下载并解压SDK:[下载链接](http://wiki.connect.qq.com/sdk下载)

2. 拖拽 TencentOpenAPI.framework 文件到Xcode⼯工程内(请勾选 Copy items if needed);或拷⻉贝SDK文件到⼯工程的物理理⽬目录下，选中⼯工程Target -> General -> Linked Frameworks and Libraries -> Add Other 然后找到对应⽂文件添加即可

![添加TencentOpenAPI](https://ws4.sinaimg.cn/large/006tNbRwly1fuxhi6l6vwj31kw0qkn7x.jpg)

3. 添加依赖库 SystemConfiguration.framework

4. 新增⼀一条URL scheme:选中⼯工程Target -> Info -> URLTypes;新的scheme命名为: tencent+appid(ex: tencent123456)

5. 添加白名单:LSApplicationQueriesSchemes新增⽩白名单，详⻅见demo
```
<key>LSApplicationQueriesSchemes</key>
	<array>
        <string>mqqOpensdkSSoLogin</string>
        <string>mqqopensdkapiV2</string>
        </array>
```

![白名单](https://ws4.sinaimg.cn/large/006tNbRwly1fuxhnmm925j31c00lwdlu.jpg)

6. Appdelegate的handleOpenURL代理理⽅方法中中添加处理理回调的代码


```
#import <TencentOpenAPI/TencentOAuth.h>
#import <TencentOpenAPI/QQApiInterface.h>
#import <TencentOpenAPI/QQApiInterfaceObject.h>


- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    
    //[QQApiInterface handleOpenURL:url delegate:(id<QQApiInterfaceDelegate>)[QQApiShareEntry class]];
    
    if (YES == [TencentOAuth CanHandleOpenURL:url])
    {
        UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Where from" message:url.description delegate:nil cancelButtonTitle:@"ok" otherButtonTitles:nil, nil];
        [alertView show];
        return [TencentOAuth HandleOpenURL:url];
    }
    return YES;
}

- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
    //[QQApiInterface handleOpenURL:url delegate:(id<QQApiInterfaceDelegate>)[QQApiShareEntry class]];
    
    if (YES == [TencentOAuth CanHandleOpenURL:url])
    {
        return [TencentOAuth HandleOpenURL:url];
    }
    return YES;
}
```
7. QQ登陆相关实现逻辑

ViewController.m 
```
#import "ViewController.h"
#import <TencentOpenAPI/QQApiInterface.h>
#import <TencentOpenAPI/TencentOAuth.h>
#import <TencentOpenAPI/QQApiInterfaceObject.h>


@interface ViewController ()<TencentSessionDelegate>
{
    TencentOAuth *_tencentOAuth;
    NSMutableArray *_permissionArray;   //权限列表
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
}

- (IBAction)qqLogin:(id)sender {
    //AppId需替换成自己申请的appid
    _tencentOAuth = [[TencentOAuth alloc] initWithAppId:@"1107812292" andDelegate:self];
    _permissionArray = [NSMutableArray arrayWithObjects:
                        kOPEN_PERMISSION_GET_SIMPLE_USER_INFO,
                        nil];
    [_tencentOAuth authorize:_permissionArray];
}

#pragma TencentSessionDelegate
/**
 * [该逻辑未实现]因token失效而需要执行重新登录授权。在用户调用某个api接口时，如果服务器返回token失效，则触发该回调协议接口，由第三方决定是否跳转到登录授权页面，让用户重新授权。
 * \param tencentOAuth 登录授权对象。
 * \return 是否仍然回调返回原始的api请求结果。
 * \note 不实现该协议接口则默认为不开启重新登录授权流程。若需要重新登录授权请调用\ref TencentOAuth#reauthorizeWithPermissions: \n注意：重新登录授权时用户可能会修改登录的帐号
 */
- (BOOL)tencentNeedPerformReAuth:(TencentOAuth *)tencentOAuth{
    return YES;
}
- (BOOL)tencentNeedPerformIncrAuth:(TencentOAuth *)tencentOAuth withPermissions:(NSArray *)permissions{
    
    // incrAuthWithPermissions是增量授权时需要调用的登录接口
    // permissions是需要增量授权的权限列表
    [tencentOAuth incrAuthWithPermissions:permissions];
    return NO; // 返回NO表明不需要再回传未授权API接口的原始请求结果；
    // 否则可以返回YES
}
-(void)tencentDidLogin{
    NSLog(@"----ok-----");
    /** Access Token凭证，用于后续访问各开放接口 */
    if (_tencentOAuth.accessToken) {
        
        //获取用户信息。 调用这个方法后，qq的sdk会自动调用
        //- (void)getUserInfoResponse:(APIResponse*) response
        //这个方法就是 用户信息的回调方法。
        
        [_tencentOAuth getUserInfo];
    }else{
        
        NSLog(@"accessToken 没有获取成功");
    }
}
//-(NSArray *)getAuthorizedPermissions:(NSArray *)permissions withExtraParams:(NSDictionary *)extraParams{
//    NSLog(@"----%@********%@-----",permissions,extraParams);
//    return permissions;
//}
- (void)getUserInfoResponse:(APIResponse*) response{
    NSLog(@"*********");
    NSLog(@" response %@",response);
    NSLog(@"*********");
}

-(void)tencentDidNotLogin:(BOOL)cancelled{
    if (cancelled) {
        NSLog(@" 用户点击取消按键,主动退出登录");
    }else{
        NSLog(@"其他原因， 导致登录失败");
    }
}
-(void)tencentDidNotNetWork{
    NSLog(@"没有网络了， 怎么登录成功呢");
}


@end
```


参考链接1 [QQ原生三方登录]https://www.jianshu.com/p/456cbc0dbf4d
