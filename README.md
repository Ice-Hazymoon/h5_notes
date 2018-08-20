# 用于记录H5开发的一些坑

## html2canvas 生成的背景图片模糊问题

> 使用 `img` 标签即可解决

## `overflow: auto|scroll` 在移动设备上卡顿的解决方法

```css
-webkit-overflow-scrolling: touch;
```

## 让Swiper移动端超出一屏的部分滚动显示而不是触发滑到下一页

> 来自[https://www.wikimoe.com/?post=68](https://www.wikimoe.com/?post=68)

```javascript
var swiper = new Swiper('.swiper-container', {
	direction: 'vertical',
});
var startScroll, touchStart, touchCurrent;
swiper.slides.on('touchstart', function (e) {
	startScroll = this.scrollTop;
	touchStart = e.targetTouches[0].pageY;
}, true);
swiper.slides.on('touchmove', function (e) {
	touchCurrent = e.targetTouches[0].pageY;
	var touchesDiff = touchCurrent - touchStart;
	var slide = this;
	var onlyScrolling =
			( slide.scrollHeight > slide.offsetHeight ) && 
			(
				( touchesDiff < 0 && startScroll === 0 ) || 
				( touchesDiff > 0 && startScroll === ( slide.scrollHeight - slide.offsetHeight ) ) || 
				( startScroll > 0 && startScroll < ( slide.scrollHeight - slide.offsetHeight ) ) 
			);
	if (onlyScrolling) {
		e.stopPropagation();
	}
}, true);
```

## 部分视频在 iOS 设备上播放没有声音

> 可能和视频的各种参数有关? 最后试了各种方法, 用迅雷影音压缩后得以解决

## canvas drawimage 绘制大图时出现锯齿

> 虽然有很多算法可以解决, 但最简单的还是使用`context.imageSmoothingQuality = "high"`, 设置采样质量, 兼容性可能不是很好, 但用了几个项目没有发现大问题

## 1px 的 base64图片

或许会用到?

```html
<img src="data:image/gif;base64,R0lGODlhAQABAIAAAAUEBAAAACwAAAAAAQABAAACAkQBADs=">
```

## html2canvas 在 iOS 上生成图片时报错

html2canvas 在 iOS 上生成图片时出现 `invalidStateError: The object is in an invalid state` 的问题

搜了半天发现貌似整个互联网就我出现了这种问题

解决方案: 人工用 cnavas 把页面画出来.....(插件坑了我两天

## canvas drawimage 绘制多张大图时 iOS 出不来的问题

类似网易 2016 娱乐圈画传的 H5 [http://ent.163.com/special/entphotos2017/](http://ent.163.com/special/entphotos2017/)

写了个类似的画中画小页面, 遇到了图片在iOS上出不来的bug, 我判断了所有图片均加载完成后将加载完的图片存入数组, 然后在 canvas 中使用可解决

判断多张图片加载完成, 代码来自: [https://stackoverflow.com/a/8682318/9040159](https://stackoverflow.com/a/8682318/9040159)

```JavaScript
// loader will 'load' items by calling thingToDo for each item,
// before calling allDone when all the things to do have been done.
function loader(items, thingToDo, allDone) {
    if (!items) {
        // nothing to do.
        return;
    }

    if ("undefined" === items.length) {
        // convert single item to array.
        items = [items];
    }

    var count = items.length;

    // this callback counts down the things to do.
    var thingToDoCompleted = function (items, i) {
        count--;
        if (0 == count) {
            allDone(items);
        }
    };

    for (var i = 0; i < items.length; i++) {
        // 'do' each thing, and await callback.
        thingToDo(items, i, thingToDoCompleted);
    }
}

function loadImage(items, i, onComplete) {
    var onLoad = function (e) {
        e.target.removeEventListener("load", onLoad);

        // this next line can be removed.
        // only here to prove the image was loaded.
        document.body.appendChild(e.target);

        // notify that we're done.
        onComplete(items, i);
    }
    var img = new Image();
    img.addEventListener("load", onLoad, false);
    img.src = items[i];
}

var items = ['http://bits.wikimedia.org/images/wikimedia-button.png',
             'http://bits.wikimedia.org/skins-1.18/common/images/poweredby_mediawiki_88x31.png',
             'http://upload.wikimedia.org/wikipedia/en/thumb/4/4a/Commons-logo.svg/30px-Commons-logo.svg.png',
             'http://upload.wikimedia.org/wikipedia/commons/3/38/Icons_example.png'];

loader(items, loadImage, function () {
    alert("done");
});
```

## 微信无法保存 BlobURL 的图片, 请使用 DataURL

```JavaScript
canvas.toDataURL('image/jpeg');
```

## iOS 微信音频自动播放

```javascript
const bgm = new Audio('bgm.mp3')
wx.ready(e=>{
	bgm.play();
})
```

## iOS 微信视频样式

> 参考 [https://x5.tencent.com/tbs/guide/video.html](https://x5.tencent.com/tbs/guide/video.html)

- x5-video-player-type: 启用Ｈ5同层播放器
- x5-video-player-fullscreen: 全屏方式
- x5-video-orientation 控制横竖屏

```html
<video id="video" style="object-fit:fill;width: 100%;height: 100%;" x5-video-player-type="h5" preload="auto" x-webkit-airplay="true" playsinline="true" webkit-playsinline="true">
    <source src="video.mp4" type="video/mp4">
</video>
```

## iOS 微信视频自动播放

```html
<video id="video" src="video.mp4"></video>
```

```javascript
wx.ready(e=>{
    video.play();
})
```

## 分享相关

> 来自: [https://github.com/wjfz/weixin-jssdk](https://github.com/wjfz/weixin-jssdk)

```php
<?php
$appId = '';
$appsecret = '';
$timestamp = time();
$jsapi_ticket = make_ticket($appId,$appsecret);
$nonceStr = make_nonceStr();
$url = 'http://'.$_SERVER['HTTP_HOST'].$_SERVER['REQUEST_URI'];
$signature = make_signature($nonceStr,$timestamp,$jsapi_ticket,$url);
function make_nonceStr(){
	$codeSet = '1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
	for ($i = 0; $i<16; $i++) {
		$codes[$i] = $codeSet[mt_rand(0, strlen($codeSet)-1)];
	}
	$nonceStr = implode($codes);
	return $nonceStr;
}
function make_signature($nonceStr,$timestamp,$jsapi_ticket,$url){
	$tmpArr = array(
        'noncestr' => $nonceStr,
        'timestamp' => $timestamp,
        'jsapi_ticket' => $jsapi_ticket,
        'url' => $url
	);
	ksort($tmpArr, SORT_STRING);
	$string1 = http_build_query( $tmpArr );
	$string1 = urldecode( $string1 );
	$signature = sha1( $string1 );
	return $signature;
}
function make_ticket($appId,$appsecret){
	// access_token 应该全局存储与更新，以下代码以写入到文件中做示例
	$data = json_decode(file_get_contents("access_token.json"));
	if ($data->expire_time < time()) {
		$TOKEN_URL="https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=".$appId."&secret=".$appsecret;
		$json = file_get_contents($TOKEN_URL);
		$result = json_decode($json,true);
		$access_token = $result['access_token'];
		if ($access_token) {
			$data->expire_time = time() + 7000;
			$data->access_token = $access_token;
			$fp = fopen("access_token.json", "w");
			fwrite($fp, json_encode($data));
			fclose($fp);
		}
	}else{
		$access_token = $data->access_token;
	}
	// jsapi_ticket 应该全局存储与更新，以下代码以写入到文件中做示例
	$data = json_decode(file_get_contents("jsapi_ticket.json"));
	if ($data->expire_time < time()) {
		$ticket_URL="https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=".$access_token."&type=jsapi";
		$json = file_get_contents($ticket_URL);
		$result = json_decode($json,true);
		$ticket = $result['ticket'];
		if ($ticket) {
			$data->expire_time = time() + 7000;
			$data->jsapi_ticket = $ticket;
			$fp = fopen("jsapi_ticket.json", "w");
			fwrite($fp, json_encode($data));
			fclose($fp);
		}
	}else{
		$ticket = $data->jsapi_ticket;
	}
	return $ticket;
}
?>
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>Document</title>
	<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=0">
</head>
<body>
	<script src="http://res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
    <script src="./js/app.js"></script>
	<script>
		wx.config({
			debug: false,
			appId: '<?=$appId?>',
			timestamp: <?=$timestamp?>,
			nonceStr: '<?=$nonceStr?>',
			signature: '<?=$signature?>',
			jsApiList: [
				'checkJsApi',
				'onMenuShareTimeline',
				'onMenuShareAppMessage'
			]
		});
		wx.ready(function () {
			var shareData = {
				title: '标题',
				desc: '摘要',
				link: '链接',
				imgUrl: '图片'
			};
			wx.onMenuShareAppMessage(shareData);
			wx.onMenuShareTimeline(shareData);
		});
		wx.error(function (res) {
			alert(res.errMsg);
		});
	</script>
</body>
</html>
```

## 其他

- Apple 部分设备的微信不兼容 ES6 (貌似和系统版本号无关), 未查明原因
