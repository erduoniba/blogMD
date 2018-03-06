### UIWebView相关知识点

#### 1、监听UIWebView的滚动以及WebView实际高度

设置下 webView.scrollerView.delegate = self。

也就是设置了UIWebView的滚动代理方法，然后再执行，代理方法走通了。

```objective-c
-(UIWebView *)showWebView
{
    if (!_showWebView) {
        _showWebView = [[UIWebView alloc] initWithFrame:CGRectMake(0, 0, SCREEN_WIDTH,SCREEN_HEIGHT)];
        _showWebView.delegate = self;
        _showWebView.scrollView.delegate = self; 
    }
    return _showWebView;
}
- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
    NSLog(@"<==== %0.2f ====>", scrollView.contentOffset.y);
}

- (void)scrollViewDidScroll:(UIScrollView *)scrollView{
	//下面这行代码是获取web view的实际高度
	NSInteger htmlheight = [[self.showWebView stringByEvaluatingJavaScriptFromString:@"document.body.scrollHeight"] integerValue];
    NSLog(@"<==== %0.2f ====>", scrollView.contentOffset.y);
}
```



#### 2、背景颜色改变

```objective-c
_webView.opaque = NO;
_webView.backgroundColor = [UIColor whiteColor];
```



#### 3、获取webview点击区域的图片

[How to get the image from uiwebview in ios](http://stackoverflow.com/questions/9703071/how-to-get-the-image-from-uiwebview-in-ios)

```objective-c
-(BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
NSLog(@"TAPPED");
//Touch gestures below top bar should not make the page turn.
//EDITED Check for only Tap here instead.
if ([gestureRecognizer isKindOfClass:[UITapGestureRecognizer class]]) {
    CGPoint touchPoint = [touch locationInView:self.view];

    NSUserDefaults * userDefaults = [NSUserDefaults standardUserDefaults];
    bool pageFlag = [userDefaults boolForKey:@"pageDirectionRTLFlag"];
    NSLog(@"pageFlag tapbtnRight %d", pageFlag);

    if(self.interfaceOrientation==UIInterfaceOrientationPortrait||self.interfaceOrientation==UIInterfaceOrientationPortraitUpsideDown) {
        NSString *imgURL = [NSString stringWithFormat:@"document.elementFromPoint(%f, %f).src", touchPoint.x, touchPoint.y];
        NSString *urlToSave = [webV stringByEvaluatingJavaScriptFromString:imgURL];
        NSLog(@"urlToSave :%@",urlToSave);
        NSURL * imageURL = [NSURL URLWithString:urlToSave];
        NSData * imageData = [NSData dataWithContentsOfURL:imageURL];
        UIImage * image = [UIImage imageWithData:imageData];
        imgView.image = image;//imgView is the reference of UIImageView
    }
  }
return YES;
}
```



#### 4、获取User-Agent及修改

[获取 UIWebview 的 Useragent，以及附加自定义字段到 Useragent](http://blog.csdn.net/mangosnow/article/details/38798195) 

User-Agent是Http协议中的一部分，属于头域的组成部分，User-Agent也简称UA。用较为普通的一点来说，是一种向访问网站提供你所使用的浏览器类型、操作系统及版本、CPU 类型、浏览器渲染引擎、浏览器语言、浏览器插件等信息的标识。UA字符串在每次浏览器 HTTP 请求时发送到服务器！浏览器UA 字串的标准格式为： 浏览器标识 (操作系统标识; 加密等级标识; 浏览器语言) 渲染引擎标识 版本信息。

通过user-agent不能完全准确的判断是属于那款浏览器。由于UA字符串在每次浏览器HTTP 请求时发送到服务器，所以服务器就可以根据它来做好多事。比如：

1、统计用户浏览器使用情况。有些浏览器说被多少人使用了，实际上就可以通过判断每个IP的UA来确定这个IP是用什么浏览器访问的，以得到使用量的数据。

2、根据用户使用浏览器的不同，显示不同的排版从而为用户提供更好的体验。有些网站会根据这个来调整打开网站的类型,如是手机的就打开wap，显示非手机的就打开pc常规页面。用手机访问谷歌和电脑访问是不一样的，这些是谷歌根据访问者的UA来判断的。 

iOS项目中一般使用一个基类的webviewVC，可以在该类的实现文件中 调用initialize类方法

```objective-c
+ (void)initialize{
    UIWebView* webView = [[UIWebView alloc] initWithFrame:CGRectZero];
    NSString* secretAgent = [webView stringByEvaluatingJavaScriptFromString:@"navigator.userAgent"];
    NSString *newUagent = [NSString stringWithFormat:@"%@ appname/customMessage",secretAgent];
    NSDictionary *dictionary = [[NSDictionary alloc] initWithObjectsAndKeys:newUagent, @"UserAgent", nil];
    [[NSUserDefaults standardUserDefaults] registerDefaults:dictionary];
}
```



##### 5、JS和iOS交互

[iOS开发之Objective-C与JavaScript的交互](http://www.cnblogs.com/zhuqil/archive/2011/08/03/2126562.html)

```objective-c
- (BOOL)webView:(UIWebView *)webView
        shouldStartLoadWithRequest:(NSURLRequest *)request
        navigationType:(UIWebViewNavigationType)navigationType
{
//点击网页上的某个商品,点击事件会获取 这个点击事件的信息，我们就可以获取我们想要的
NSString *urlString = [[request URL] absoluteString];  //http://bh.ewt.cc/showitem/19675.htm
        // 从http://bh.ewt.cc/showitem/19675.htm 截取 showitem/19675.htm
        NSArray *arr = [urlString componentsSeparatedByString:@"http://bh.ewt.cc/"];
        
        if (arr.count > 1) {
            NSString *productIdInfo = arr[1];
            
            // 从 showitem/19675.htm 截取 showitem/19675
            NSArray *productIdArr = [productIdInfo componentsSeparatedByString:@"."];
            if (productIdArr.count > 1) {
                NSString *p = productIdArr[productIdArr.count - 2];
                
                // 从 showitem/19675 截取 19675
                NSArray *pA = [p componentsSeparatedByString:@"/"];
                if (pA.count > 1) {
                    NSString *productID = pA[1];
                    ProductDetailViewController *proDVC = [ProductDetailViewController shareInstance];
                    proDVC.productId = productID;
                    [self.navigationController pushViewController:proDVC animated:YES];//跳到原生界面
                    
                    return NO;
                }
            }
        }
}
```