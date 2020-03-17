## Redis

### Docker

```shell
docker run -di -p 6379:6379 --name redis redis --requirepass "969023014"
```

```yml
spring:
	redis:
    host: 120.77.176.55
    port: 6379
    password: 123456
```



redis模拟用于降级处理的预警

```java
private Object saveOrderFail(HttpServeletRequest request){
  	String saveOrderKey = "save-order";
  	String sendValue = redisTemplate.opsForValue().get(saveOrderKey);
  	final String ip = request.getRemoteAddr();
  	new Thread(()->{
      	if(StringUtils.isBlank(sendValue)) {
          	System.out.println("用户下单失败,IP地址为":ip);
          	redisTemplate.opsForValue().set(saveOrderKey,"save-order-fail",20,TimeUnit.SECONDS);
        }else{
          	System.out.println("20s内不要重复发送短信");
        }
    }).start();
}
```

### 企业级解决方案

#### 缓存预热

系统启动前，提前将相关的缓存数据直接加载到缓存系统，避免用户在请求时，先查询数据库，然后将数据缓存。用户直接查询实现被预热的缓存数据。

#### 缓存雪崩

在一个较短时间内，缓存中较多的key集中过期。Redis未命中，向数据库获取数据，数据库同时接受大量请求无法及时处理，Redis大量请求积压，数据库崩溃，重启后缓存中仍然无数据可用。

解决办法

- 页面静态化
- 多级缓存
- 灾难预警机制
- 限流降级

- 数据有效期调整
- LRU与LFU切换
- 超热数据使用永久Key

#### 缓存击穿

某个超热key过期，对数据库服务器造成压力。Redis正常。

#### 缓存穿透

非正常URL访问，Redis无法命中，服务器无法查询到数据。

解决方法

- 缓存null
- 白名单策略
- 实时监控
- key加密