## 相册搬到了自己的hexo博客

上面的那些页面代码没在用了，使用这个md文件的教程就可以了

<a target="_blank" href="https://me.idealli.com/photos/" class="LinkCard">
兰州小红鸡的博客相册
</a>

昨天突发奇想给博客写了个相册页面，使用腾讯云cos作为相册的存储桶，使用api在线获取相册里面的存储桶里的照片并且实时生成相册内容。之前也有看过一些人做的相册页面，但是对于我来说，还是感觉不方便。网上的大多是在本地项目文件夹存放照片，然后更改一系列的主题文件来实现相册页面。

比如这位制作的[Hexo NexT 博客增加瀑布流相册页面](https://blog.dongleizhang.com/posts/3720dafc/)，然而他做的过程已经算是比较简洁了，没有改动太多主题配置。但是这种相册，每次添加新照片的时候，还是需要手动在相册页面添加相应的图片链接与代码。

所以我想能不能做一个直接后台上传图片，不用再改动代码的静态博客的相册页面呢，就像一个动态博客一样，或者像qq相册那样，只需要上传照片就可以了。

答案是可以的，机智的我使用了腾讯云的cos存储桶作为相册后台，调用cos存储桶的xml文件api在线获取图片链接，再使用JavaScript代码动态生成相册内容。

步骤如下：

<!--more-->

## 创建腾讯云cos存储桶

这个比较简单，搜索腾讯云，注册账号登陆，在云产品中选择对象存储，新建一个存储桶。就OK了。

![兰州小红鸡](https://image.idealli.com/blog/18122202.jpg)

### 跨域访问cors设置

在基础配置中找到cors设置

![兰州小红鸡](https://image.idealli.com/blog/18122201.jpg)

操作选择GET，来源Origin填写你的域名，带http或者https，其他默认不要填，如下图

![兰州小红鸡](https://image.idealli.com/blog/18122203.jpg)

然后记住这个地方**访问域名**，这里就是我们动态生成相册，获取链接时需要用到的xml链接，下面要用到

![兰州小红鸡](https://image.idealli.com/blog/18122204.jpg)


然后就ok了

## hexo本地配置

在本地项目新建一个相册页面

```
$ hexo new page photos
```

编辑`photos.md`文件，写入以下代码

<p class="wraning">记得在下面的代码中填写xmllink的值，也就是上面提到的你的存储桶访问域名</p>

```html
<style type="text/css">
	.main-inner{
		width: 100%;
	}
	.main {
    padding-bottom: 150px;
    margin-top: 0px;
	}
	.main-inner{
		margin-top: unset;
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
	.site-meta .brand{
		border-radius: 3px;
		background: rgba(255,255,255,0.5);
	}
	.imgbox{
	 padding: 0px;
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
    height: 600px;
	}
	.box li span{
	display: block;
    padding: 4% 7% 10% 7%;
    min-height: 180px;
    background: #fff;
    border-right: 1px solid #bcbcbc;
    color: #393939;
    font-size: 21px;
    font-family: 'SwisMedium';
    line-height: 26px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
	}
	img.imgbox{
		padding: unset;
		border: unset;
		position: relative;
	}
	img{
		padding: unset;
	}

	div#comments.comments.v {
    border: 0px;
    margin: auto !important;
    margin-top: unset;
    margin-left: unset;
    margin-right: unset;
    width: 60%;
    padding-top: 50px;
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
    background: #fff;
    border-right: 1px solid #bcbcbc;
    color: #393939;
    font-size: 12px;
    font-family: 'SwisMedium';
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    text-align: center;
}

.posts-expand .post-title {
	display: none;
}
.btn-more-posts{
    display: inline-block;
    vertical-align: middle;
    zoom: 1;
    background: url(https://image.idealli.com/bg11.jpg);
    font: 85px/250px 'ChaletComprimeMilanSixty';
    background-position: right bottom;
    color: #fff;
    background-size: 100%;
    width: 100%;
    text-align: center;
    border: unset;
    height: 1000px;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
}


@media (max-width: 767px){
	.box li {
    width: 100%;
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
</style>

<div id="box" class="box"></div>
<button class="btn-more-posts" id="more">
	<span style="display: inline;">
		<strong style="color: #f0f3f6;">LOAD MORE</strong>
	</span>
</button>

<script type="text/javascript">
if (window.XMLHttpRequest)
{// code for IE7+, Firefox, Chrome, Opera, Safari
    xmlhttp=new XMLHttpRequest();
}
else
{// code for IE6, IE5
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}

var xmllink="你的访问域名链接"
//访问域名链接就是我上面提到的那个访问域名xml链接

xmlhttp.open("GET",xmllink,false);
xmlhttp.send();
xmlDoc=xmlhttp.responseXML;
var urls=xmlDoc.getElementsByTagName('Key');
var date=xmlDoc.getElementsByTagName('LastModified');

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
	item.innerHTML="<img class=imgbox src="+xmllink+bucket+" width=550 height=550><span>"+bucket+"</span><p>"+date[i].innerHTML.substring(0,10)+"</p>";
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
	item.innerHTML="<div class=btn-more-posts><div style=display:inline;><strong style=color:#f0f3f6; >"+bucket.substring(0,length)+"</strong></div></div>";
	box.appendChild(item);
	continue;
	}
		var item=document.createElement("li");
		item.innerHTML="<img class=imgbox src="+xmllink+urls[i].innerHTML+" width=550 height=550><span>"+urls[i].innerHTML+"</span><p>"+date[i].innerHTML.substring(0,10)+"</p>";
	box.appendChild(item);
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
