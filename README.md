## 相册搬到了自己的hexo博客

**上面的那些页面代码没在用了，使用这个md文件的教程就可以了**


![兰州小红鸡](https://image.idealli.com/blog/18122304.jpg)

昨天突发奇想给博客写了个相册页面，使用腾讯云cos作为相册的存储桶，使用api在线获取相册里面的存储桶里的照片并且实时生成相册内容。之前也有看过一些人做的相册页面，但是对于我来说，还是感觉不方便。网上的大多是在本地项目文件夹存放照片，然后更改一系列的主题文件来实现相册页面。

比如这位制作的[Hexo NexT 博客增加瀑布流相册页面](https://blog.dongleizhang.com/posts/3720dafc/)，然而他做的过程已经算是比较简洁了，没有改动太多主题配置。但是这种相册，每次添加新照片的时候，还是需要手动在相册页面添加相应的图片链接与代码。

所以我想能不能做一个直接后台上传图片，不用再改动代码的静态博客的相册页面呢，就像一个动态博客一样，或者像qq相册那样，只需要上传照片就可以了。

<!--more-->

答案是可以的，机智的我使用了腾讯云的cos存储桶作为相册后台，调用cos存储桶的xml文件api在线获取图片链接，再使用JavaScript代码动态生成相册内容。


<a target="_blank" href="https://me.idealli.com/photos/" class="LinkCard">
兰州小红鸡的博客相册
</a>

**效果如下**

前方轻微秀恩爱预警

![](https://image.idealli.com/blog/2019050708.png)

![](https://image.idealli.com/blog/2019050706.png)

![](https://image.idealli.com/blog/2019050707.png)

![](https://image.idealli.com/blog/2019050705.png)

> **2019-05-07更新**
之前写的这篇教程没想到有这么多朋友采纳
不过教程也确实有很多问题
之前写的比较粗糙，让朋友们遇到各种麻烦
现在更新一下
写一下我能想到的各种bug的解决方法

步骤如下：

## 创建腾讯云cos存储桶

这个比较简单，搜索腾讯云，注册账号登陆，在云产品中选择对象存储，新建一个存储桶。就OK了。

![兰州小红鸡](https://image.idealli.com/blog/18122202.jpg)

### 跨域访问cors设置

在基础配置中找到cors设置

![兰州小红鸡](https://image.idealli.com/blog/18122201.jpg)

操作选择GET，来源Origin填写你的域名，带http或者https，其他默认不要填，如下图

![兰州小红鸡](https://image.idealli.com/blog/18122203.jpg)

**注意**：如果填了域名还是遇到跨域问题，那么就把`origin`源填为`*`，比如下面这样（其实刚开始弄的话建议都填成`*`好了）

![](https://image.idealli.com/blog/2019050703.png)

### 读写权限

一般情况下默认是共有读私有写，policy权限就不要设置了

![](https://image.idealli.com/blog/2019050704.png)

### 访问域名

然后记住这个地方**访问域名**，这里就是我们动态生成相册，获取链接时需要用到的xml链接，下面要用到

![兰州小红鸡](https://image.idealli.com/blog/18122204.jpg)

复制这个访问域名，看能不能在浏览器中打开，如果可以打开并且没有显示error节点，那么就可以继续下面的操作，否则查看上面哪一步出错并进行改正

### 上传照片

上传照片方式有很多（推荐用coscmd命令行写个脚本），不过这里教程中你就直接上腾讯云后台上传就好了，想要其他骚操作等相册做好了再自己百度吧。

**重要事项**

1. 上传照片前，先在存储桶中建立一个文件夹，也就是你的相册名字，当然你也可以新建多个文件夹。
2. 但是有一点需要需要注意的是，**不能直接上传一个文件夹**，那样会出bug，见完文件夹后往里面上传照片，文件夹里面不能再新建文件夹了（除非你自己改造下面的相应代码）
3. 每个文件中需要一张命名为**封面**的图片，它会作为你的该文件夹相册的封面



然后就ok了

## hexo本地配置

在本地项目新建一个相册页面

```
$ hexo new page photos
```
<p class="note">
	19-01-15更新：将照片地展示分不同相册加载
</p>

编辑\\source\\photos\\路径下的`index.md`文件，写入以下代码

<p class="wran">记得在下面的代码中填写xmllink的值，也就是上面提到的你的存储桶访问域名</p>

```html
<style type="text/css">
	.main-inner{
		width: 100%;
	}
	.main {
    padding-bottom: 150px;
    margin-top: 0px;
    background: #121212;
	}
	.main-inner{
		margin-top: unset;
	}
	.page-post-detail .post-meta{
		display: none;
	}
	body {
		background-image: unset;
		background-attachment: unset;
		background-size: 100%;
		/*background-position: top left;*/
	}
	.header{
		background: rgba(28, 25, 25, 0.6);
		border-bottom: unset;
	}
	.menu .menu-item a{
		    font-weight: 300;
    		color: #e6eaed;
	}

	.imgbox{
	 width: 100%;
	 overflow: hidden;
	 height: 250px;
	 border-right: 1px solid #bcbcbc;
	}
	.box{
		visibility: visible;
		overflow: auto; 
		zoom: 1;
	}
	.box li{
	float: left;
    width: 25%;
    position: relative;
    overflow: hidden;
    text-align: center;
    list-style: none;
    margin: 0;
    /*display: inline;*/
    padding: 0;
    height: 360px;
	}
	.box li span{
	display: block;
    padding: 4% 7% 10% 7%;
    min-height: 80px;
    background: #fff;
    color: #fff;
    font-size: 16px;
    background: #121212;
    font-weight: 600;
    line-height: 26px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
	}

	img.imgitem{
		padding: unset;
		padding: unset;
		border: unset;
		position: relative;
		padding: 0px;
		height: auto;
		width: 100%;
	}


div#posts.posts-expand {
    border: unset;
    padding: unset;
    margin-bottom: 10px;
}
.posts-expand .post-body img{
	padding: 0px !important;
}
.box p{
	display: block;
    background: #121212;
    color: #fff;
    font-size: 12px;
    font-family: 'SwisMedium';
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    text-align: center;
}

.box span strong{
	background: rgba(0,0,0,0.4);
	padding: 20px;
}

.posts-expand .post-title {
	display: none;
}
.title{
    display: inline-block;
    vertical-align: middle;
    background: url(https://image.idealli.com/bg11.jpg);
    font: 85px/250px 'ChaletComprimeMilanSixty';
    background-position: left bottom !important;
    color: #fff;
    background-size: 100% auto !important; 
	-webkit-background-size: cover; 
	-moz-background-size: cover;
	-o-background-size: cover;
    width: 100%;
    text-align: center;
    border: unset;
    height: 700px;
    cursor: unset !important;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
}
.btn-more-posts{
	display: inline-block;
    vertical-align: middle;
    font: 85px/250px 'ChaletComprimeMilanSixty';
    color: #000;
    width: 100%;
    text-align: center;
    border: unset;
    height: 400px;
    background-color: #121212;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
}

@media (max-width: 767px){
	.box li {
    width: 100%;
}
.title {
    height: 200px;
}

.box span {
    min-height: 80px;
    border-right: unset;
    font-size: 17px;
}
.box p{
    border-right: unset;
    font-size: 12px;
  
}
.posts-expand {
    margin: unset;
}
	div#comments.comments.v {
    width: 96%;
    padding-top: 50px;
}


}

@media (min-width: 1600px){
	.container .main-inner{
		width: 100%;
	}
}

.footer{
	background-color: #121212 !important;
}
.v * {
    color: #f4f4f4 !important;
}

.v .vwrap .vmark .valert .vcode {
    background: #00050b !important;
}

</style>

<div id="box" class="box"></div>


<script type="text/javascript">

function loadXMLDoc(xmlUrl) 
{
	try //Internet Explorer
	{
		xmlDoc=new ActiveXObject("Microsoft.XMLDOM");
	}
	catch(e)
	{
	  try //Firefox, Mozilla, Opera, etc.
	    {
		  xmlDoc=document.implementation.createDocument("","",null);
	    }
	  catch(e) {alert(e.message)}
	}
	
	try 
	{
		  xmlDoc.async=false;
		  xmlDoc.load(xmlUrl);
	}
	catch(e) {
		try //Google Chrome  
		  {  
			var chromeXml = new XMLHttpRequest();
			chromeXml.open("GET", xmlUrl, false);
			chromeXml.send(null);
			xmlDoc = chromeXml.responseXML.documentElement; 				
			//alert(xmlDoc.childNodes[0].nodeName);
			//return xmlDoc;    
		  }  
		  catch(e)  
		  {  
			  alert(e.message)  
		  }  		  	
	}
	return xmlDoc; 
}

var xmllink="你的访问域名链接"
//访问域名链接就是我上面提到的那个访问域名xml链接

xmlDoc=loadXMLDoc(xmllink);
var urls=xmlDoc.getElementsByTagName('Key');
var date=xmlDoc.getElementsByTagName('LastModified');
var wid=250;
var showNum=12; //每个相册一次展示多少照片
if ((window.innerWidth)>1200) {wid=(window.innerWidth*3)/18;}
var box=document.getElementById('box');
var i=0;

var content=new Array();
var tmp=0;
var kkk=-1;
for (var t = 0; t < urls.length ; t++) {
	var bucket=urls[t].innerHTML;
	var length=bucket.indexOf('/');
	if(length===bucket.length-1){
		kkk++;
		content[kkk]=new Array();
		content[kkk][0]={'url':bucket,'date':date[t].innerHTML.substring(0,10)};
		tmp=1;
	}
	else {
		content[kkk][tmp++]={'url':bucket.substring(length+1),'date':date[t].innerHTML.substring(0,10)};
	}
}

for (var i = 0; i < content.length; i++) {
	var conBox=document.createElement("div");
	conBox.id='conBox'+i;
	box.appendChild(conBox);
	var item=document.createElement("div");
	var title=content[i][0].url;
	item.innerHTML="<button class=title style=background:url("+xmllink+'/'+title+"封面.jpg"+");><span style=display:inline;><strong style=color:#f0f3f6; >"+title.substring(0,title.length-1)+"</strong></span></button>";
	conBox.appendChild(item);

	for (var j = 1; j < content[i].length && j < showNum+1; j++) {
		var con=content[i][j].url;
		var item=document.createElement("li");
		item.innerHTML="<div class=imgbox id=imgbox style=height:"+wid+"px;><img class=imgitem src="+xmllink+'/'+title+con+" alt="+con+"></div><span>"+con.substring(0,con.length-4)+"</span><p>上传于"+content[i][j].date+"</p>";
		conBox.appendChild(item);
	}
	if(content[i].length > showNum){
		var moreItem=document.createElement("button");
		moreItem.className="btn-more-posts";
		moreItem.id="more"+i;
		moreItem.value=showNum+1;
		let cur=i;
		moreItem.onclick= function (){
			moreClick(this,cur,content[cur],content[cur][0].url);
		}
		moreItem.innerHTML="<span style=display:inline;><strong style=color:#f0f3f6;>加载更多</strong></span>";
		conBox.appendChild(moreItem);
	}
}

function moreClick(obj,cur,cont,title){
	var parent=obj.parentNode;
	parent.removeChild(obj);
	var j=obj.value;
	var begin=j;
	for ( ; j < cont.length && j < Number(showNum) + Number(begin); j++) {
		console.log( Number(showNum) + Number(begin));
		var con=cont[j].url;
		var item=document.createElement("li");
		item.innerHTML="<div class=imgbox id=imgbox style=height:"+wid+"px;><img class=imgitem src="+xmllink+'/'+title+con+" alt="+con+"></div><span>"+con.substring(0,con.length-4)+"</span><p>上传于"+cont[j].date+"</p>";
		parent.appendChild(item);
	}
	if(cont.length > j){
		obj.value=j;
		parent.appendChild(obj);
	}
}

</script>
```

然后刷新hexo渲染

```
$ hexo clean
$ hexo d -g
```

再往cos存储桶里上传照片，就可以了！效果如下
重要的是以后更新照片都不用改动代码。往腾讯云cos上传照片就好了，而且腾讯云页提供了很多工具可以再本地命令行上传照片，非常方便，感兴趣可以自行百度。

![兰州小红鸡](https://image.idealli.com/blog/18122205.jpg)

嘻嘻女朋友真可爱！

![兰州小红鸡](https://image.idealli.com/blog/18122206.jpg)


## 自己写界面样式

**注意**

> 以上教程是基于我自己的博客主题（next主题Mist样式）做的适配
所以可能会出现一些样式问题
强烈建议会前端的朋友自己写样式和逻辑代码，确保不会出错。
不会前端的朋友也可以学一学css自己改改，挺有趣的
另外都在搞这个了，应该也都会一点了
自己写css样式也不麻烦，一个小时就能把相册改造成自己喜欢的风格

### 代码核心

上面的photo界面的核心代码是获取腾讯云xml文件的那段

```javascript
function loadXMLDoc(xmlUrl) 
{
	try //Internet Explorer
	{
		xmlDoc=new ActiveXObject("Microsoft.XMLDOM");
	}
	catch(e)
	{
	  try //Firefox, Mozilla, Opera, etc.
	    {
		  xmlDoc=document.implementation.createDocument("","",null);
	    }
	  catch(e) {alert(e.message)}
	}
	
	try 
	{
		  xmlDoc.async=false;
		  xmlDoc.load(xmlUrl);
	}
	catch(e) {
		try //Google Chrome  
		  {  
			var chromeXml = new XMLHttpRequest();
			chromeXml.open("GET", xmlUrl, false);
			chromeXml.send(null);
			xmlDoc = chromeXml.responseXML.documentElement; 				
			//alert(xmlDoc.childNodes[0].nodeName);
			//return xmlDoc;    
		  }  
		  catch(e)  
		  {  
			  alert(e.message)  
		  }  		  	
	}
	return xmlDoc; 
}

var xmllink="你的访问域名链接"
//访问域名链接就是我上面提到的那个访问域名xml链接

xmlDoc=loadXMLDoc(xmllink);
var urls=xmlDoc.getElementsByTagName('Key');
```

`urls`便是获取了相册的所有链接，之后要做的事情就是用js动态生成img元素
我的是这样写的，当然你也可以使用别的写法，写适合自己风格的相册样式

```javascript
var urls=xmlDoc.getElementsByTagName('Key');
var date=xmlDoc.getElementsByTagName('LastModified');
var wid=(window.innerWidth*3)/18;
var box=document.getElementById('box');
var i=0;

for ( ; i < 21 && i<urls.length; i++) {
	var bucket=urls[i].innerHTML;
	var length=bucket.indexOf('/');
	if(length===bucket.length-1){
	var item=document.createElement("div");
	item.innerHTML="<button class=btn-more-posts><span style=display:inline;><strong style=color:#f0f3f6; >"+bucket.substring(0,length)+"</strong></span></button>";
	box.appendChild(item);
	continue;
	}

	var item=document.createElement("li");
	item.innerHTML="<div class=imgbox id=imgbox style=height:"+wid+"px;><img class=imgitem src="+xmllink+'/'+bucket+" ></div><span>"+bucket+"</span><p>"+date[i].innerHTML.substring(0,10)+"</p>";
	box.appendChild(item);

}

var morep=document.getElementById('more');
morep.onclick=function(){
	var temp=i+20;

	for ( ;  i < temp && i<urls.length; i++) {
	var bucket=urls[i].innerHTML;
	var length=bucket.indexOf('/');
	if(length===bucket.length-1){
	var item=document.createElement("div");
	item.innerHTML="<button class=btn-more-posts><span style=display:inline;><strong style=color:#f0f3f6; >"+bucket.substring(0,length)+"</strong></span></button>";
	box.appendChild(item);
	continue;
	}
		var item=document.createElement("li");
		item.innerHTML="<div class=imgbox id=imgbox style=height:"+wid+"px;><img class=imgitem src="+xmllink+'/'+urls[i].innerHTML+" ></div><span>"+urls[i].innerHTML+"</span><p>"+date[i].innerHTML.substring(0,10)+"</p>";
	box.appendChild(item);

	}
}

```

整个教程还是提供了一个思路，如果遇到问题了，请根据自己的实际情况进行
我的博客主题：Next主题Mist样式

**注意事项**

1. 上传照片前，先在存储桶中建立一个文件夹，也就是你的相册名字，当然你也可以新建多个文件夹。
2. 但是有一点需要需要注意的是，**不能直接上传一个文件夹**，那样会出bug，见完文件夹后往里面上传照片，文件夹里面不能再新建文件夹了（除非你自己改造下面的相应代码）
3. 每个文件中需要一张命名为**封面**的图片，它会作为你的该文件夹相册的封面
4. 确保存储桶的xml域名能在浏览器上访问
5. 暂时不要设置policy权限啥的，等你相册制作完了，如果怕照片被盗可以添加这些东西
6. 尽量自己修改样式，不然肯定不好看（除非你的主题和我的一样）
7. css样式直接写在md文件中，不用写在全局里

---

<p class="note">
	下面是一些我的建站笔记汇总，平常做的小手工，希望对大家有帮助
</p>

<a target="_blank" href="https://me.idealli.com/post/e8d13fc.html" class="LinkCard">
hexo博客搭建以及next美化教程
</a>

<a target="_blank" href="https://me.idealli.com/post/ed80a662.html" class="LinkCard">
原生js实现网页图片点击展示效果
</a>

<a target="_blank" href="https://me.idealli.com/post/2d5da13e.html" class="LinkCard">
用回valine评论系统,valine评论框样式美化
</a>

<a target="_blank" href="https://me.idealli.com/post/73ad4183.html" class="LinkCard">
给hexo静态博客添加动态相册功能
</a>

<a target="_blank" href="https://me.idealli.com/post/6bf81741.html" class="LinkCard">
hexo建站笔记之首页文章轮播图
</a>

<a target="_blank" href="https://me.idealli.com/post/a714f04b.html" class="LinkCard">
模仿知乎的链接卡片
</a>

<a target="_blank" href="https://me.idealli.com/post/fb598031.html" class="LinkCard">
将公众号文章爬到hexo博客
</a>

<a target="_blank" href="https://me.idealli.com/post/1609338289.html" class="LinkCard">
使用腾讯云cdn加速博客
</a>

<a target="_blank" href="https://me.idealli.com/post/d6caa003.html" class="LinkCard">
hexo建站笔记之彩色标签云
</a>

<a target="_blank" href="https://me.idealli.com/post/eccf2c93.html" class="LinkCard">
hexo建站笔记之带图标的标签云
</a>
