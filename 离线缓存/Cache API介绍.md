# 5.2. Cache API介绍

caches是window的属性之一，用于缓存静态资源或请求返回的数据。

主要的缓存数据来源：<br/>
1.Service Sorker的install事件中缓存静态资源，比如缓存html、css、js等静态资源<br/>
2.Service Worker的fetch事件中缓存请求的数据<br/>
3.用户交互请求的数据，比如查看某些图片、视频等<br/>

使用caches之前，需要判断浏览器是否支持：

```
if('caches' in window){
    //支持
}
```

**Caches API：**

1.创建或打开cache

如果缓存中存在cacheName则直接打开，不存在则创建新的cacheName。

```
caches.open(cacheName)  //自定义cacheName
```

返回一个promise，对cacheName的存储、删除等操作需要在返回的promise中进行。

2.缓存数据

（1）`add(url)`：缓存单个静态数据，参数为静态数据路径。路径错误，则缓存失败。

```
caches.open(cacheName).then(cache => {
    cache.add('./index.js')
})
```

（2）`allAll(urlArray)`：缓存多个静态数据，参数为静态数据路径的数组。如果数组中某一项的路径错误，整个数组的静态数据缓存失败。

```
caches.open(cacheName).then(cache => {
    cache.add([
        '/',
        './index.html',
        './js/index.js',
        './css/index.css',
    ])
})
```

（3）`put(request,response)`：缓存请求的数据，第一个参数为请求的url，第二个参数为请求返回数据。

```
fetch(url).then(response => {
    cache.put(url,response)
}) 
```

3.查找缓存

缓存的查找是根据cacheName中的请求进行查找。

（1）`match(request,options)` : 查找第一个匹配的缓存。

第一个参数为需要匹配的请求，第二个参数为请求的过滤：

`ignoreSearch` ：Boolean值，默认值为false，当设置为true，则过滤掉hash，如请求为`http://xxx.com?q=xxx`将会被过滤掉。<br/>
`ignoreMethod` ：Boolean值，默认值为false，当设置为true，则阻止对request请求的http方法的验证（通常只允许GET和HEAD两种请求方法）。<br/>
`ignoreVary` ：Boolean值，默认值为false，当设置为true，则忽略对VARY头信息的匹配。如当请求的request匹配成功，对与获取的response值，不会进行VARY头信息的匹配。<br/>
`cacheName` ：缓存名，一般忽略。

如果不匹配，则返回undefined。匹配成功，则返回带有reponse的promise。

```
caches.open(cacheName).then(cache => {
    cache.match('./index.js').then(function(response){
        //response为返回结果
    })
})
```

（2）`matchAll(request,options)` ：查找全部匹配的缓存。参数和返回结果同`match`。

4.删除缓存 

`delete(key)` : 删除流程为找到匹配的缓存并删除，返回的结果为promise。

```
caches.open(cacheName).then(function(cache) {
  cache.matchAll('./images/').then(function(response) {
    response.forEach(function(element, index, array) {
      cache.delete(element);
    });
  });
})
```

5.遍历缓存

遍历所有的缓存,参数同match，非必参，当为传入参数，则返回所有的缓存。

```
caches.keys(request,options).then(function(keys){
    //返回的keys为数组
})
```