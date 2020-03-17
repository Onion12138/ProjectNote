## Docker

运行MongoDB并且设置密码认证

```shell
docker run -di --name mongodb -p 27017:27017 -v ~/notehub/mongodb:/data/db --restart=always mongo --auth
 
docker exec -it mongodb mongo admin #mongodb为容器名

db.createUser({ user: 'admin', pwd: 'admin', roles: [ { role: "root", db: "admin" } ] }); 
```

