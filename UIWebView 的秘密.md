#### UIWebView 的秘密

##### 1、UIWebView 的秘密－第一部分 Xmlhttprequest上的内存泄漏

原文地址：[UIWebView Secrets - Part1 - Memory Leaks on Xmlhttprequest](http://blog.techno-barje.fr/post/2010/10/04/UIWebView-secrets-part1-memory-leaks-on-xmlhttprequest/) 

我的第一个博客主要揭示使用 `UIWebView` 时出现的大的内存泄漏。`UIWebView` 是 `iPhone` 上唯一展示 `HTML` 的控件。`UIWebView` 有很多不同的问题，我来说一下最大的问题点。实际上，所有的 `XMLHttpRequests` 使用 `javascript` 代码是完全泄漏的！我的意思是当你请求了检索100ko（[knockoutjs](http://www.cnblogs.com/wujilong/p/3396268.html)  简称 ko）的数据时 ,你的内存使用量便会增长到100ko！这个bug并不总是活跃，但是大多数会这样，事实上，简单的打开一个简单的链接就会触发 `UIWebView` ，比如点击<https://github.com>这个链接。

但是当我们执行 [simple test application](http://blog.techno-barje.fr/public/iphone-sdk/UIWebViewLeaks.zip) 这个项目时来看看它的内存使用图：

![](http://blog.techno-barje.fr/public/iphone-sdk/profile-xmlhttprequest-0-then-1-labeled.png) 

1.建立  `UIWebView` 对象

2.加载本地的 `HTML` 测试文件

3.运行项目，执行3次 XMLHttpRequest 请求, 注意每个请求后是如何释放三倍内存！

4.引发泄漏原因是通过打开一个页面,该页面重定向回我们的测试文件

5.执行相同的3次 XMLHttpRequest 请求,看看有多少内存使用和完全泄露

6.我们用

```objective-c
[webview stringByEvaluatingJavaScriptFromString:@"document.body.innerHTML='';"];
```

来清理 `HTML` (当我们有很多DOM对象，有时释放会一些内存)

7.释放UIWebView(几乎没有内存释放, 在下一篇文章会分析)




所以,综上所述,通常情况下,当你在 `UIWebView` 执行这个Javascript :

```javascript
var xmlhttp = new XMLHttpRequest();
xmlhttp.onreadystatechange = function() {
	if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
		xmlhttp.onreadystatechange = null;
		onSuccess(xmlhttp);
		xmlhttp.abort();
	}
};
xmlhttp.open("GET", "http://your.domain/your.request/...", true);
xmlhttp.send();	
```

你将占用并泄漏一个大量的内存!

但有一个hack来解决这个问题:当你打开一个链接时修复它。事实上,这导致泄漏是应用程序设置 的*WebKitCacheModelPreferenceKey* 属性。当你使用 `UIWebView` 打开一个链接,这个属性会自动设置为“1”的值。那么,解决方案是每次你打开一个链接后将改值设置回0。你可以很容易地通过在 `UIWebView` 的 代理`UIWebViewDelegate` 添加如下代码:

```objective-c
- (void)webViewDidFinishLoad:(UIWebView *)webView {
    [[NSUserDefaults standardUserDefaults] setInteger:0
                                         forKey:@"WebKitCacheModelPreferenceKey"];
}
```

修改之后你的程序将减少因为低内存而导致 程序crash。
