# 5.3.1. 注册Service worker

使用Service Worker第一步，创建serviceWorker.js，并在js中注册Service Worker

```javascript
//index.js
//判断是否支持serviceWorker
if('serviceWorker' in navigator){
    //serviceWorker.js后缀为随机戳，方便更新serviceWorker.js
    navigator.serviceWorker.register('./serviceWorker.js', { scope : '/'}).then(function(){
        console.log('service worker注册成功');
    })
}
```

使用`navigator.serviceWorker.register(url,scope)`注册Service Worker，第一个参数为`serviceWorker.js`的地址，第二个参数`scope`为缓存文件的范围，最大范围是`serviceWorker.js`所在的当前目录，范围的设定只能为`serviceWorker.js`同目录或子目录，比如`serviceWorker.js`的路径为`app/public/`，子文件有`img`，范围可以设定为同目录`/`或子目录`/img/`，不能设为`app/static/`等非同目录和子目录，因此需要将`serviceWorker.js`放在需要缓存的文件的最外层路径里。当第二个参数为空时，默认是范围是同目录范围，即`/`。