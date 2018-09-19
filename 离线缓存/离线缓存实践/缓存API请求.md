# 5.3.3. 缓存API请求

除了静态资源，还可以缓存动态请求的数据，动态请求的数据与静态数据的差异在于如果本地缓存中有请求的缓存，在服务器请求未返回前使用缓存的数据，服务器请求返回后，更新数据。

在demo中的缓存搜索新闻的请求：

1.缓存搜索api的数据

在fetch事件中缓存动态请求的数据

```javascript
//serviceWorker.js
var fetchCacheName = 'news-api-v1', //唯一的缓存名
    cacheFetchUrls = [      //缓存的api
        '/news?'
    ];

self.addEventListener('fetch' , function(e){
    //判断请求是否需要缓存
    var needCache = cacheFetchUrls.some(function(url){
        return e.request.url.indexOf(url) > 1;
    })
    if(needCache){
        caches.open(fetchCacheName).then(function(cache){
            return fetch(e.request).then(function(response){
                //缓存请求
                if(response.statusText !== 'Not Found'){
                    cache.put(e.request.url,response.clone())
                }
                return response;
            })
        })
    }
})
```

由于内存效率问题，请求的response流只能读一次，如果直接执行`cache.put(request, response)`将response流存入内存中，将不能返回给浏览器，因此需要将response复制再存储。详细介绍可参考[What happens when you read a response?](https://jakearchibald.com/2014/reading-responses/)

2.从缓存中读取

```javascript
//index.js
getDataFromCache : function(url){
    if('caches' in window){
        return caches.match(url).then(function(cache){
            if(!cache || cache.responseText == 'Not Found'){
                return;
            }
            //如果缓存中有匹配的请求，返回数据
            return cache.json();
        })
    }else{
        return Promise.resolve();
    }
}
```

封装从缓存中搜索匹配的请求，如匹配成功，返回缓存的数据。

3.请求数据

请求数据，当数据未返回时，优先展示缓存中的数据，数据返回后，当新数据和缓存数据不同时，重新更新数据，并缓存新数据。

```javascript
//index.js
queryNews : function(){
    var _this = this,
        newsName = document.querySelector('#input').value;
        url = '/news?q=' + newsName;    //搜索新闻接口
    
    if(newsName === ''){
        alert('请输入想要了解的新闻');
        return;
    }

    var fetchData = _this.fetchNewsApi(url);        //fetch搜索新闻接口
    var cacheData;
    
    //缓存中匹配请求
    _this.getDataFromCache(url).then(function(data){
        //匹配成功
        if(data && data.data &&  data.data.length > 0){
            _this.fillNews(data.data)
        }
        cacheData = data || {};
        return fetchData;
    }).then(function(data){
        //请求成功，重新渲染数据
        if(JSON.stringify(data) !== JSON.stringify(cacheData)){
            _this.fillNews(data.data)
        }
    }).catch(function(err){
        console.log(err);
    }) 
}
```

当断网后，就可以继续搜索同样的新闻～