#### 解决并发冲突一般两种常用方式

##### ①用UI组件自带的loading效果，阻止重复操作

##### axios拦截器统一处理
###### 目标
```
只发第一个请求。在 A 请求还处于 pending 状态时，后发的所有与 A 重复的请求都取消，实际只发出 A 请求，直到 A 请求结束（成功/失败）才停止对这个请求的拦截。
```

###### 思路
```
①将项目中所有的 pending 状态的请求存储在一个变量中，叫它 pendingRequests；把每个请求的方法、url 和参数组合成一个字符串，作为标识该请求的唯一 key，同时也是 pendingRequests 对象的 key；
②在请求发出前检查当前请求是否重复(有：cancel 掉当前请求；没有：把 requestKey 添加到 pendingRequests 对象中)
③在请求返回后维护 pendingRequests 对象
④需要清空 pendingRequests 对象的场景
```

###### 关键代码
```
let pendingRequests = new Map()

const requestKey = `${config.url}/${JSON.stringify(config.params)}/${JSON.stringify(config.data)}&request_type=${config.method}`;

// 请求拦截器
axios.interceptors.request.use(
  (config) => {
    if (pendingRequests.has(requestKey)) {
      config.cancelToken = new axios.CancelToken((cancel) => {
        // cancel 函数的参数会作为 promise 的 error 被捕获
        cancel(`重复的请求被主动拦截: ${requestKey}`);
      });
    } else {
      pendingRequests.set(requestKey, config);
      config.requestKey = requestKey;
    }
    return config;
  },
  (error) => {
    // 这里出现错误可能是网络波动造成的，清空 pendingRequests 对象
    pendingRequests.clear();
    return Promise.reject(error);
  }
);

// 响应拦截器
axios.interceptors.response.use((response) => {
  const requestKey = response.config.requestKey;
  pendingRequests.delete(requestKey);
  return Promise.resolve(response);
}, (error) => {
  if (axios.isCancel(error)) {
    console.warn(error);
    return Promise.reject(error);
  }
  pendingRequests.clear();
  return Promise.reject(error);
})

// 页面切换时清空
router.beforeEach((to, from, next) => {
  request.clearRequestList();
  next();
});
```