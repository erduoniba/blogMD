### shell中的curl网络请求

>[curl](https://curl.haxx.se/) 是利用URL语法在命令行下工作的文件传输工具，1997年首次发行，支持文件上传和下载，结合shell脚本体验更棒。但按照传统习惯称 `curl`  为下载工具。
>
> `curl` 支持的通信协议有 有[FTP](https://zh.wikipedia.org/wiki/FTP)、[FTPS](https://zh.wikipedia.org/wiki/FTPS)、[HTTP](https://zh.wikipedia.org/wiki/HTTP)、[HTTPS](https://zh.wikipedia.org/wiki/HTTPS)、[TFTP](https://zh.wikipedia.org/wiki/TFTP)、[SFTP](https://zh.wikipedia.org/wiki/SFTP) 等等，支持的平台有 Linux、MacOSX、Darwin、Windows、DOS、FreeBSD等等。

#### 一、curl的作用：

**1、查看网页源码**

```shell
denglibingdeMacBook-Pro-4: curl www.baidu.com

<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=http://s1.bdstatic.com/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn"></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=http://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');</script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>

// 保存整个网页，使用 -o 处理
denglibingdeMacBook-Pro-4: curl -o baidu www.baidu.com
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2381  100  2381    0     0  77899      0 --:--:-- --:--:-- --:--:-- 79366
```



**2、查看头信息**

```shell
denglibingdeMacBook-Pro-4: denglibing$ curl -i www.baidu.com
HTTP/1.1 200 OK
Server: bfe/1.0.8.18
Date: Mon, 03 Jul 2017 09:12:17 GMT
Content-Type: text/html
Content-Length: 2381
Last-Modified: Mon, 23 Jan 2017 13:28:11 GMT
Connection: Keep-Alive
ETag: "588604eb-94d"
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Pragma: no-cache
Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
Accept-Ranges: bytes
...
...
...
```



**3、发送网络请求信息**

GET方式请求：

```Shell
curl example.com/form.cgi?data=xxx  如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
```

POST方式请求：

```shell
//数据和网址分开，需要使用 '--data' 或者 '-d' 参数; curl默认使用GET，使用 '-X' 参数可以支持其他动词， 更多的参数使用 'man curl' 查看
$ curl -X POST --data "data=xxx" example.com/form.cgi

// '--user-agent' 字段，表表面客户端的设备信息：
$ curl --user-agent "Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_2 like Mac OS X) AppleWebKit/603.2.4 (KHTML, like Gecko) Mobile/14F89/mideaConnect MissonWebKit/4021/zh-Hans (AppStore) (4347701760)" http://www.example.com

//使用 '--cookie' 参数，可以让curl发送cookie
$ curl --cookie "name=xxx" www.example.com

//添加头信息,自行增加一个头信息。'--header' 或者 '-H' 参数就可以起到这个作用
$ curl --header "Content-Type:application/json" http://example.com


//提交文件作为请求信息 使用 '@文件名' 请求
$ curl -X POST -H "Content-Type: text/xml" -d @denglibing.txt http://example.com

//denglibing.txt:
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ser="http://services.webservices.com"><soapenv:Header/><soapenv:Body><ser:addEmpIdCardrecord><ser:empId>11622695,D58C6A25-C683-47D6-A18C-B7741284F632</ser:empId></ser:addEmpIdCardrecord></soapenv:Body></soapenv:Envelope>
```



#### 二、实例

```shell
denglibingdeMacBook-Pro-4:~ denglibing$ curl https://api.github.com/users
[
  {
    "login": "mojombo",
    "id": 1,
    "avatar_url": "https://avatars3.githubusercontent.com/u/1?v=3",
    "gravatar_id": "",
    "url": "https://api.github.com/users/mojombo",
    "html_url": "https://github.com/mojombo",
    "followers_url": "https://api.github.com/users/mojombo/followers",
    "following_url": "https://api.github.com/users/mojombo/following{/other_user}",
    "gists_url": "https://api.github.com/users/mojombo/gists{/gist_id}",
    "starred_url": "https://api.github.com/users/mojombo/starred{/owner}{/repo}",
    "subscriptions_url": "https://api.github.com/users/mojombo/subscriptions",
    "organizations_url": "https://api.github.com/users/mojombo/orgs",
    "repos_url": "https://api.github.com/users/mojombo/repos",
    "events_url": "https://api.github.com/users/mojombo/events{/privacy}",
    "received_events_url": "https://api.github.com/users/mojombo/received_events",
    "type": "User",
    "site_admin": false
  }
]
```

当然，既然这些请求是在命令行中执行，完全可以写成shell脚本，来处理一系列的工作，比如多个请求，而shell脚本在Mac中，可以使用定时任务触发，进而完成一系列的自动化工作。



#### 三、相关链接

[curl网站开发指南](http://www.ruanyifeng.com/blog/2011/09/curl.html)

[How do I POST XML data with curl](https://stackoverflow.com/questions/2063520/how-do-i-post-xml-data-with-curl) 