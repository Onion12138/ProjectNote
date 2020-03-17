## Mongodb



### 文档操作

#### 聚合



### 文档设计

#### 一对多&多对多

在Mongodb中可以通过数组的方式实现一对多和多对多，不需要关联表。导致的问题则是如果关联表经常变化，会导致更新操作很麻烦。使用lookup实现类似关系数据库的join操作。

```
Contacts
	name: "Onion",
	company: "Apple",
	group_ids: [1,2,3]
	
Groups
	group_id: 1
	name: "Friends"
	
db.contacts.aggregate([
		{
				$lookup:
				{
						from: "groups",
						localField: "group_ids",
						foreignField: "group_id",
						as: "groups"
				}
		}
])
```

注意事项：

- 无主外键检查
- 只支持left outer join
- from不能为分片表

#### 列转行

```json
{
		title: "Dunkirk",
  	...
  	release_USA: "2017/07/23",
  	release_UK: "2017/08/01"
}
```

文档中有很多类似字段上述的设计不便添加索引。

更好的文档设计

db.movies.createIndex({"releases.country":1,"releases.date":1})

```json
{
		title: "Dunkirk",
  	...
  	releases: [
      {country:"USA",date:"2017/07/23"},
      {country:"UK",date:"2017/08/01"}
    ]
}
```

#### 版本字段

新增字段导致前后文档字段不一致，可以增加版本字段，利用版本字段筛选

#### 近似计算

统计流量，数据不需要特别精确，减少数据写入

伪码如下

```
if random(0,9) == 0
		increment by 10
```

#### 预聚合

场景：热销榜、电影排行

传统解决方案，聚合计算时间长

```json
{
		product:"Bike",
		sku:"abc12138",
  	quantity: 20394,
		daily_sales: 40,
		weekly_sales: 302,
		monthly_sales: 1419
}
```

```json
db.inventory.update({_id:123},
{
		$inc: {
				quantity: -1,
				daily_sales: 1,
				weekly_sales: 1,
				monthly_sales: 1
		}
})
```

### 事务

### 索引

ESR原则

E:Equal

S:Sort

R:Range

### 复制



### 备份

```
mongodump --host localhost:27017 --oplog
mongorestore --host localhost:27017 --oplogReplay
```





### Docker环境部署

```shell
docker run -di --name mongodb -p 27017:27017 -v ~/notehub/mongodb:/data/db --restart=always mongo --auth
 
docker exec -it mongodb mongo admin #mongodb为容器名

db.createUser({ user: 'admin', pwd: 'admin', roles: [ { role: "root", db: "admin" } ] }); 
```

### 与SpringBoot整合

```yml
spring:
  data:
    mongodb:
      host: 182.61.23.125
      database: notehub
      username: admin
      password: admin
      authentication-database: admin
```

