#前言--回答这个问题是一个作业，不过据说也是前端面试必答题。我在网上看到这篇文章觉得，写的很好，为了加深理解顺便锻炼下蹩脚的英文能力，就决定把他翻译下了。
原文链接[http://igoro.com/archive/what-really-happens-when-you-navigate-to-a-url/](http://igoro.com/archive/what-really-happens-when-you-navigate-to-a-url/)
  作为一个软件开发者，你一定对web应用的工作流程和其所包含的技术，例如浏览器、HTTP、HTML、web server、request handlers等有着很深的理解。
    这篇文章，我们将会深入了解下当我们访问一个URL时背后所发生事件的顺序。
    # 1. 你在浏览器输入一个URL
    它开始于此：
    ![](http://upload-images.jianshu.io/upload_images/4655976-9c18a772c84b1954.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    # 2. 浏览器查找主域名的IP地址
    ![](http://upload-images.jianshu.io/upload_images/4655976-63669ef5948cc686.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    ## 导航的第一步就是查找出你要访问的域名的IP地址。DNS查找过程如下：
    - **浏览器缓存（Browser cache）**——浏览器会缓存DNS记录一段时间。有趣的是操作系统不会告诉浏览器，这些DNS记录要缓存多长时间，所以浏览器对于这些缓存都有一个固定的缓存时间（不同浏览器不同，2-30分钟）。
    - **操作系统缓存（OS cache）**——如果浏览器缓存不包含要找的DNS记录，那么浏览器就会请求操作系统，要操作系统从本地的host文件夹中查找。
    - **路由器缓存（Router cache）**——请求会继续上升到路由器。（路由器通常也有DNS缓存）
    - **ISP DNS缓存（ISP DNS cache）**—— 接下来要查找的地方就是你的域名服务器上的DNS缓存
    - **递归搜索（Recursive search）**——你的域名服务器开始从根域名服务器递归查找，通过.com顶级域名服务器到Facebook的域名服务器。通常DNS服务器缓存里有这些.com域名服务器的名字，所以对根域名服务器的访问不是必须的。

    下面是一张DNS递归查找的示意图：
    ![](http://upload-images.jianshu.io/upload_images/4655976-1df3b6b4d6ebfa18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    # 3. 浏览器向web服务器发送一个HTTP请求
    ![](http://upload-images.jianshu.io/upload_images/4655976-935ea44f11ad6689.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    可以非常确定我们从浏览器缓存中不能得到Facebook主页，因为动态页面过期很快或者立即就过期（他们把过期日期设置在过去）。
    所以，浏览器将会发送这个请求给Facebook服务器：
    ```
    GET http://facebook.com/ HTTP/1.1
    Accept: application/x-ms-application, image/jpeg, application/xaml+xml, [...]
    User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; [...]
    Accept-Encoding: gzip, deflate
    Connection: Keep-Alive
    Host: facebook.com
    Cookie: datr=1265876274-[...]; locale=en_US; lsd=WW[...]; c_user=2101[...]
    ```
    GET request 命令URL提取地址：http://facebook.com/ 。
    浏览器标识自身（User-Agent header），并且指定它将会接受哪些类型的响应(Accept and Accept-Encoding headers)。这个connection header要求服务器保持TCP链接开放以便接受进一步的请求。
    # 4. Facebook服务器以重定向（permanent redirect）响应
    ![](http://upload-images.jianshu.io/upload_images/4655976-db91246033d71abe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    下面是Facebook服务器返回给浏览器请求的响应：
    ```
    HTTP/1.1 301 Moved Permanently
    Cache-Control: private, no-store, no-cache, must-revalidate, post-check=0,
          pre-check=0
	  Expires: Sat, 01 Jan 2000 00:00:00 GMT
	  Location: http://www.facebook.com/
	  P3P: CP="DSP LAW"
	  Pragma: no-cache
	  Set-Cookie: made_write_conn=deleted; expires=Thu, 12-Feb-2009 05:09:50 GMT;
	        path=/; domain=.facebook.com; httponly
		Content-Type: text/html; charset=utf-8
		X-Cnection: close
		Date: Fri, 12 Feb 2010 05:09:51 GMT
		Content-Length: 0
		```
		服务器以一个**301 Moved Permanently response**回复浏览器，告诉他跳转*http://www.facebook.com/*而不是*http://facebook.com/*
		# 5. 浏览器跟随重定向
		![](http://upload-images.jianshu.io/upload_images/4655976-b580d1878a159dfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
		浏览器现在知道*http://www.facebook.com/*是正确的地址，所以它发送另外一条GET request:
		```
		GET http://www.facebook.com/ HTTP/1.1
		Accept: application/x-ms-application, image/jpeg, application/xaml+xml, [...]
		Accept-Language: en-US
		User-Agent: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; [...]
		Accept-Encoding: gzip, deflate
		Connection: Keep-Alive
		Cookie: lsd=XW[...]; c_user=21[...]; x-referer=[...]
		Host: www.facebook.com
		```
		头部的信息和前面那条请求是一样的。
		# 6. 服务器处理请求
		![](http://upload-images.jianshu.io/upload_images/4655976-5c1bdd3b7e959f63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
		浏览器收到GET请求，处理它然后返回一个响应。
		这个过程看起来简单明了，但其实其中包含了很多有趣的东西——即便是在一个像我博客样简单的网站，更不用说是在一个大规模可扩展的网站，如Facebook。
		## - Web服务器软件
		Web服务器软件（如IIS、Apache）接收HTTP请求，决定哪个请求处理程序（在ASP.NET,PHP,Ruby,...）应该被执行去处理这个请求。
		## - 请求处理程序（Request handler）
		请求处理程序读取请求并且生成HTML响应。
		# 7. 服务器发回HTML响应
		下面是服务器产生并返回的响应：
		```
		HTTP/1.1 200 OK
		Cache-Control: private, no-store, no-cache, must-revalidate, post-check=0,
		    pre-check=0
		    Expires: Sat, 01 Jan 2000 00:00:00 GMT
		    P3P: CP="DSP LAW"
		    Pragma: no-cache
		    Content-Encoding: gzip
		    Content-Type: text/html; charset=utf-8
		    X-Cnection: close
		    Transfer-Encoding: chunked
		    Date: Fri, 12 Feb 2010 09:05:55 GMT

		    2b3
		    ��������T�n�@����[...]
		    ```
		    这个** Content-Encoding header**告诉浏览器响应正文以gzip算法压缩。解压blob后，可以看到希望的HTML：
		    ```
		    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
		          "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
			  <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en"
			        lang="en" id="facebook" class=" no_js">
				<head>
				<meta http-equiv="Content-type" content="text/html; charset=utf-8" />
				<meta http-equiv="Content-language" content="en" />
				...
				```
				除哪种压缩外，**header**还会指定是否缓存页面，通过那种方式缓存，cookies如何设置，隐私如何处理等等。
				# 8. 浏览器开始渲染HTML
				甚至在浏览器接收完整个HTML文档之前，它就开始渲染网站了。
				![](http://upload-images.jianshu.io/upload_images/4655976-218184299bc313f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
				# 9. 浏览器为嵌入对象发送请求
				![](http://upload-images.jianshu.io/upload_images/4655976-961a983e9556a544.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
				当浏览器渲染HTML时，它会注意那些需要发送请求另外一个地址的标签。浏览器会发送GET请求，获取相应文件。（一些图片，CSS，js、、、）
				下面是我访问facebook时，浏览器获取的文件URL：
				## - Images
				http://static.ak.fbcdn.net/rsrc.php/z12E0/hash/8q2anwu7.gif
				http://static.ak.fbcdn.net/rsrc.php/zBS5C/hash/7hwy7at6.gif
				…
				## -  CSS style sheets
				http://static.ak.fbcdn.net/rsrc.php/z448Z/hash/2plh8s4n.css
				http://static.ak.fbcdn.net/rsrc.php/zANE1/hash/cvtutcee.css
				…
				## - JavaScript files
				http://static.ak.fbcdn.net/rsrc.php/zEMOA/hash/c8yzb6ub.js
				http://static.ak.fbcdn.net/rsrc.php/z6R9L/hash/cq2lgbs8.js
				…
				每个请求都会和请求HTML页面一样，走相似的流程。所以，浏览器会在DNS中查找域名，发送请求URL，重定向等等。
				当然，静态文件是可以让浏览器缓存的。一些文件可以直接从缓存中获得，不用连接服务器。浏览器知道这些特有文件缓存多长时间，因为响应头的Expires关键字设置了这个文件的有效期。除此之外，每一个响应可能也会保留一个ETag头，它就像版本号一样。如果浏览器发现这个文件的ETag号已经存在了，它可以立即停止请求。
				# 10. 浏览器进一步发送AJAX请求
				![](http://upload-images.jianshu.io/upload_images/4655976-8259d863e1f87f49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
				在web 2.0的时代，客户端会在页面渲染完成之后继续和服务器交互。
				例如，Facebook聊天室会持续更新你好友列表的在线情况。为了更新你的在线好友列表， 正在你浏览器上执行的JavaScript不得不发送一个asynchronous请求给服务器。asynchronous请求是一个以编程方式构造的GET或POST请求，并转到特殊URL。在Facebook示例中，客户端向http://www.facebook.com/ajax/chat/buddy_list.php 发送POST请求，以获取在线的朋友的列表。
				这种模式有时被称为“AJAX”，它代表“Asynchronous JavaScript And XML”，为什么服务器必须将响应格式化为XML没有特别的原因。例如，Facebook响应Asynchronous请求返回JavaScript代码片段。
				—————————————————————
				第一篇翻译处女座，还有很多需要改进的地方，比如一些专有名词的翻译和使用，大家看着有什么疑问，尽量戳原文地址[http://igoro.com/archive/what-really-happens-when-you-navigate-to-a-url/](http://igoro.com/archive/what-really-happens-when-you-navigate-to-a-url/)。
				翻译的时候也参考了google和前人的一篇译文，地址戳：[http://blog.csdn.net/tangxiaolang101/article/details/54670218](http://blog.csdn.net/tangxiaolang101/article/details/54670218)

				#**本文章著作权归饥人谷_西子和饥人谷所有，转载须说明来源！**
