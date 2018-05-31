# 用于记录微信开发的一些坑

> 以下代码均经过测试

## iOS 微信音频自动播放

```javascript
wx.ready(function(){
    audio.play();
})
```

## iOS 微信视频样式

```javascript
<video id="video" style="object-fit:fill;width: 100%;height: 100%;" x5-video-player-type="h5" preload="auto" x-webkit-airplay="true" playsinline="true" webkit-playsinline="true">
    <source src="video.mp4" type="video/mp4">
</video>
```

## iOS 微信视频自动播放

```javascript
wx.ready(function(){
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