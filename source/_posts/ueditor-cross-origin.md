---
title: Ueditor 跨域的问题研究
date: 2017-3-13
tags: 蓝桥杯
---
# Ueditor 跨域的问题研究
### 一、背景
这段时间在做CMS毕业设计项目，项目是nodejs后台，angular+require前台
前后端完全分离，采用ajax来获取数据渲染到前台上，所以前端不用任何后台语言，纯html/css/js，我将前端放在了80端口，后端放在3008端口，实际上属于两个完全不同域名。
<!--more-->
### 二、问题
首先是官方没有给nodejs的版本、然后就是这篇文章的主要问题跨域。

### 三、解决方案
UE官方给的源码都是前后端在一起，我下载了一个PHP版本的，文件结构如下：
![这里写图片描述](http://img.blog.csdn.net/20170323132937542?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG9kZDkxOTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中PHP文件夹是服务端代码，因为我用的node，所以走你~删之~~
其余的我们放入前端相关地方，然后引入之即可，需要引入的文件有三

 1. ueditor.all.(min.)js  主文件不用说
 2. ueditor.config.js  配置文件
 3. third-party/zeroclipboard/ZeroClipboard.min.js
 4. lang下的语言文件

接着，要配置UE的Home，serverUrl，我直接在ueditor.config.js 用绝对路径写死了 然后就可以开心愉快的在页面里使用啦
![这里写图片描述](http://img.blog.csdn.net/20170323133559398?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG9kZDkxOTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

但是，图片上传呢，服务端呢，翻了官网插件，哎呦，找到一个[ue for nodejs](https://github.com/netpi/ueditor)果断用之，翻阅其源码，就是处理了一下文件的保存，然后通知前端是否保存成功之类的事情。

```
var ueditor = require("./ueditor");//注意这里的‘./’ 因为我改了他的代码，所以直接把他的lib/index.js拷出来做为自己的文件用了原版只要直接require("ueditor")就好了(当然前提是你是通过npm install ueditor安装的)
	app.use("/ue", ueditor(path.join(__dirname, 'public/upload/'), function(req, res, next) {
		// ueditor 客户发起上传图片请求

		if (req.query.action === 'uploadimage') {

			// 这里你可以获得上传图片的信息
			var foo = req.ueditor;
			// console.log(foo.filename); // exp.png
			// console.log(foo.encoding); // 7bit
			// console.log(foo.mimetype); // image/png

			// 下面填写你要把图片保存到的路径 （ 以 path.join(__dirname, 'public') 作为根路径）
			var img_url = req.session.userName;
			res.ue_up(img_url); //你只要输入要保存的地址 。保存操作交给ueditor来做
		}
		//  客户端发起图片列表请求
		else if (req.query.action === 'listimage') {
			var dir_url = req.session.userName; // 插件作者:要展示给客户端的文件夹路径 笔者:这里我用了用户的用户名作为文件夹名字，请读者自行改之
			res.ue_list(dir_url) // 客户端会列出 dir_url 目录下的所有图片
		}
		// 客户端发起其它请求
		else {

			res.setHeader('Content-Type', 'application/json');
			// 这里填写 ueditor.config.json 这个文件的路径
			res.jsonp(require('./ueditor.config.json'));
		}
	}));
```
这是后台的服务端，小插曲，UE有两个配置文件，一个是前端的，还有后端的配置文件所以后端，我们只需要用jsonp把下图中的config.json暴露出来就可以，这个文件在 UE官方源码的后台代码文件夹中，这里就是PHP文件夹
![这里写图片描述](http://img.blog.csdn.net/20170323135326991?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG9kZDkxOTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里有个坑，Json标准是不支持注释的，所以用node去解析的时候会出现问题，没办法，我只好把注释全删了(当然做了个备份)

搞定这些之后，现在前端不上传图片就不会报错了，但是人类的欲望是无尽的，我们又怎么能停步于此呢，我们要上传图片！！！不管是单张的(比较坑),还是多张的(非常坑)。
首先讲一下为什么这两个区分开来，我们来看官方的这段话，这段话原地址请点[这儿](http://fex.baidu.com/ueditor/#dev-crossdomain)
![这里写图片描述](http://img.blog.csdn.net/20170323140109746?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG9kZDkxOTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

坑爹有木有，为了兼容低版本浏览器，单图提交用了iframe，而iframe不支持跨域读取内容，完了，我们的欲望要得不到满足了，(┬＿┬)↘ 跌，但是官方说要发挥想象力，网上就有人说了，**b域名的ue，上传图片到a域名，a域名保存图片成功后，重定向到b域名，在给个参数，b域名后台做处理后在返回就好了。** 坑爹，可是我的项目前端没有后台呀！！！那怎么办，然后我就发挥想象力，想象力你懂得！a域名把消息封装成参数，然后重定向到静态页面，静态页面通过js在body onload的时候把消息写到页面上去，就除了如下两个代码
#### a域名后台代码:

```
res.ue_up = function(img_url) {
          var tmpdir = path.join(os.tmpdir(), path.basename(filename));
          var name = snowflake.nextId() + path.extname(tmpdir);
          var dest = path.join(static_url, img_url, name);
          var writeStream = fs.createWriteStream(tmpdir);
          file.pipe(writeStream);
          writeStream.on("close", function() {
            fse.move(tmpdir, dest, function(err) {
              if (err) throw err;
              var obj = {
                'url': host + "upload/" + path.join(img_url, name).replace(/\\/g, '/'),//注意，这里的host是后台服务器地址，因为上传的文件存到a域名下了，所以这里地址也改了。
                'title': req.body.pictitle,
                'original': filename,
                'state': 'SUCCESS'
              }

              res.redirect(webAddress + "ue.html?obj=" + encodeURIComponent(JSON.stringify(obj)));//我改了这个地方，将obj封装了之后作为参数重定向到webAddress也就是前端地址的ue.html页面上去
            });
          })
        };
```
上边这些代码在ueditor for node 的那个lib/index.js里，也就是我的ueditor.js里，

b域名 ue.html：

```
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8"> 
<script type="text/javascript">
//获取参数的方法，网上找的
	function getUrlParam(name) {
		var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)"); //构造一个含有目标参数的正则表达式对象
		var r = window.location.search.substr(1).match(reg); //匹配目标参数
		if (r != null) return unescape(r[2]);
		return null; //返回参数值
	}
	//写参数到页面注意要在onload时调用
	var  getdata=function() {
		document.getElementsByTagName('body')[0].innerHTML=getUrlParam('obj');
	}
</script>
</head>
<body onload="getdata();">
	
</body>
</html>
```
这样就完成了单图片上<del>床囧rz=З</del> 传的功能

#### 让后我们来说说多图 Orz
多图上传的原理官方没有解释，我来解释，多图上传通过弹出一个dialogs来实现各种操作，相关文件都在dialogs里，查看dialogs/image/image.html我们发现上传用了webupload，而webupload是通过自己构建ajax来实现图片上传的，唯一有个问题，就是关于跨域session/cookie的问题，UE和webupload都没有说到，要在image.html文件所引入的image.js文件的365行左右初始化webupload配置的时候加上`withCredentials:true,`参数如下图![这里写图片描述](http://img.blog.csdn.net/20170323142039165?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG9kZDkxOTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 否则a域名的后台就得不到session了
当然这里有个小插曲，ajax的跨域需要后台headers配合，下边是我的配置

```
app.use('/', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", config.cros==="*"?req.headers.origin:config.cros);
    res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS');
    res.header("Access-Control-Allow-Headers", "X-Requested-With,x_requested_with");
    res.header("Access-Control-Allow-Credentials","true");//跨域session
    if(req.method=="OPTIONS") res.sendStatus(200);/*让options请求快速返回*/
    else  next();
});
```
上边为什么在method==options的时候要返回200，详情可以了解 [CORS preflight](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)
行了，接下来可以愉快的多图上传了，
![这里写图片描述](http://img.blog.csdn.net/20170323142912253?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRG9kZDkxOTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
什么鬼，跨域访问不允许重定向，我**，单图要重定向，多图ajax不能重定向，算了，只能在后台判断一下是不是ajax，

```
if (req.xhr||req.headers['x_requested_with']==='XMLHttpRequest') {
                res.jsonp(obj);//如果是ajax则直接jsonp
              } else {
                res.redirect(webAddress + "ue.html?obj=" + encodeURIComponent(JSON.stringify(obj)));//如果是表单提交，也就是单图的iframe提交，则重定向
              }
```
只此，UE的单图和多图上传在前后端完全分离的跨域情况下完全解决，这是我的方案，在nodejs下，同理可以推广到其他后台语言

PS:ie8下听说会用flash提交图片，我不知道如何让flash提交带上session/cookie，所以可能在ie8下后台会丢失session/cookie，未经测试。