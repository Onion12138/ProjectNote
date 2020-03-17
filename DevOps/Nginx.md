## Nginx

## 安装

```shell
yum install epel-release -y
yum list add | grep nginx
yum install nginx -y
rpm -ql nginx
/usr/sbin/nginx
```

### 配置参数

| 参数             | 含义                          |
| ---------------- | ----------------------------- |
| --prefix         | 指定安装目录                  |
| --user           | 运行nginx的worker子进程的属主 |
| --group          | 运行nginx的worker子进程的属组 |
| --pid-path       | 存放进程运行pid文件的路径     |
| --conf-path      | 配置文件的存放路径            |
| --error-log-path | 错误日志存放路径              |
| --http-log-path  | 访问日志存放路径              |
| --with-pcre      | pcre库存放路径                |
| --with-zlib      | zlib库存放路径                |

有with的默认不内置，有without的默认内置

### 组成

- 二进制可执行文件
- Nginx.conf
- access.log
- error.log

### 命令行

```shell
nginx -s reload
#帮助 -h -？
#指定配置文件 -c
#指定配置指令 -g
#指定运行目录 -p
#发送信号 -s 立刻停止：stop；优雅停止：quit，重载配置文件：reload 重新开始记录日志：reopen
#测试配置文件语法 -t -T
#打印版本 -v -V

#热部署举例
cp nginx nginx.old
cp -r nginx /usr/local/openresty/nginx/sbin/ -f
kill -USR2 13195 #nginx master process id，使用ps -ef | grep nginx 查看
kill -WINCH 13195

#日志切割
mv geek.access.log bak.log
nginx -s reopen

```

### 搭建静态资源web服务器

```nginx
log_format main '$remote_addr - $remote_user[$time_local] "$request" '
gzip on;
gzip_min_length 1;
gzip_comp_level 2;
gzip_types: text/plain application/xml;
server {
	listen 8080;
  server_name geek.taohui.pub;
  access_log logs/geek.access.log main;
  location / {
    alias dlib/; #dlib目录下存储静态资源
    autoindex on; #显示目录结构
    set $limit_rate 1k; #每秒传输1k字节
  }
}
```

### 搭建反向代理服务

```nginx
#之前nginx修改
server {
	listen 127.0.0.1:8080;
  access_log logs/geek.access.log main;
  location / {
    alias dlib/; #dlib目录下存储静态资源
    autoindex on; #显示目录结构
    set $limit_rate 1k; #每秒传输1k字节
  }
}
#新nginx
upstream local {
  server 127.0.0.1:8080
}
server {
  server_name geek.taohui.pub;
  listen 80;
  location / {
    proxy_set_header Host $host
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://local;
  }
}
```

### 信号管理

> `Master`进程

监控`worker`进程：`CHLD`

接收信号：

- `TERM`，`INT`
- `QUIT`
- `HUP`
- `USR1`
- **`USR2`**
- **`WINCH`**

```shell
kill -s SIGTERM 26733 //终止nginx
kill -s SIGHUP 26747 //生成新的worker进程，重新读取配置文件
```

> `Worker`进程

设置`worker`进程个数：

`worker_processes auto`  

接收信号：

- `TERM`，`INT`
- `QUIT`
- `USR1`
- `WINCH`

> `Nginx`命令行

- `reload`：`HUP`
- `reopen`：`USR1`
- `stop`：`TERM`
- `quit`：`QUIT`

### reload

1. 向`master`进程发送`HUP`信号
2. `master`进程校验配置语法是否正确
3. `master`进程打开新的监听端口
4. `master`进程用新配置启动新的`worker`子进程
5. `master`进程向老`worker`子进程发送`QUIT`信号
6. 老`worker`进程关闭监听句柄，处理完当前连接后结束进程

### 热升级

1. 将旧`Nginx`文件替换为新的，注意备份
2. 向`master`进程发送`USR2`信号
3. `master`进程修改`pid`文件名，加上后缀`.oldbin`
4. `master`进程用新`Nginx`文件启动新`master`进程
5. 向老`master`进程发送`WINCH`信号，关闭老`worker`
6. 回滚：向老`master`发送`HUP`，向新`master`发送`QUIT`

```shell
//旧master进程id为27394，新master进程id为27457
cp nginx nginx.bak
kill -s SIGUSER2 27394
kill -s SIGWIHCH 27394
kill -s SIGQUIT 27394

//回滚
cp nginx nginx.bak
kill -s SIGUSER2 27394
kill -s SIGWIHCH 27394
kill -s SIGHUP 27394
kill -s SIGQUIT 27457
```

### 优雅关闭worker进程

1. 设置定时器`worker_shutdown_timeout`

2. 关闭监听句柄

3. 关闭空闲连接

4. 在循环中等待全部连接关闭

5. 退出进程

### 配置

指令

- 值指定：存储配置项的值，可以合并
- 动作类指令：指定行为，不可以合并

存储值的指令集成规则：向上覆盖

- 子配置不存在时，直接使用父配置块
- 子配置存在时，直接覆盖父配置块

### 语法模块

`main`

```nginx
user nginx nginx
pid /opt/nginx/logs/nginx.pid #指定master主进程pid文件存放路径
worker_rlimit_nofile 20480 #worker进程可以打开的最大文件句柄数
worker_rlimit_core 50M #worker异常终止后的core文件
working_directory /opt/nginx/tmp #目录需要对nginx有写权限
worker_processes 4 #auto:按照CPU个数自动计算
worker_cpu_affinity 0001 0010 0100 1000 #4核4个worker子进程，绑定CPU与子进程，降低缓存失效，提升性能
worker_cpu_affinity 01 10 01 10 #2核4个worker子进程
worker_priority -10 #Linux默认优先级为120，值越小越优先，nice设定范围为-20到+19
worker_shutdown_timeout 5s #优雅退出的超时时间
worker_resolution 100ms#计时器精度，间隔越大，系统调用越少，越有利于性能提升
daemon on #后台，off为前台。前台调试，后台生产
lock_file logs/nginx.lock #锁目录
```

`event`

```nginx
worker_connections 1024 #默认为1024，可以配置为65535/worker_processes
accept_mutex on #负载均衡锁
accept_mutex_delay 200ms #必须accept_mutex为on，默认500ms
multi_accept on #是否可以连接多个
```

`server_name`

```nginx
server {
  server_name www.imooc.com
	server_name *.imooc.com
	server_name www.imooc.*
	server_name ～^www\.imooc\.*$
}
#优先级从大到小：精确匹配、左通配符匹配、右通配符匹配、正则匹配
```

### 

```nginx
location /picture {
	root /opt/nginx/html/picture;
}
#客户端请求www.test.com/picture/1.jpg，映射到/opt/nginx/html/picture/picture/1.jpg
#如果为相对路径，则是相对输入nginx -V之后--prefix=/usr/share/nginx的路径

location /picture {
	alias /opt/nginx/html/picture/; #结尾必须有/
}
#客户端请求www.test.com/picture/1.jpg，映射到/opt/nginx/html/picture/1.jpg

```

| 匹配规则 | 含义                   | 示例                           | 优先级 |
| -------- | ---------------------- | ------------------------------ | ------ |
| =        | 精确匹配               | location=/images/{...}         | 1      |
| ~        | 正则匹配，区分大小写   | location ~\\.(jpg\|gif)${...}  | 3      |
| ~*       | 正则匹配，不区分大小写 | location ~*\\.(jpg\|gif)${...} | 4      |
| ^~       | 匹配到即停止搜索       | location ^~/images/{...}       | 2      |
| 无符号   |                        | location / {...}               | 5      |

```nginx
location /test {
	#将test作为目录，如果目录不存在，尝试将test作为文件
}
location /test/ {
  #将test作为目录
}
```



```nginx
location /monitor_status {
	stub_status; #监控页面
}
#访问API：xxx/monitor_status
```

### 

```nginx
http{
	limit_conn_zone $binary_remote_addr zone=limit_addr:10m;
  server {
    location / {
      limit_conn_status 503;
  		limit_conn_log_level warn;
  		limit_conn limit_addr 2;
  		limit_rate 150;
    }
  }
}
```



```nginx
http{
	limit_conn_zone $binary_remote_addr zone=limit_addr:10m;
  limit_req_zone $binary_remote_addr zone=limit_req:15m rate=2r/m;
  server {
    location / {
      error_log logs/limit_req_error.log info;
      limit_req_status 504;
      limit_req_log_level notice;
      limit_req zone=limit_req;
      #limit_req zone=limit_req burst=7 nodelay;
    }
  }
}
```

限定IP访问

```nginx
location / {
  deny 192.168.1.1;
  allow 192.168.1.0/24;
  deny all;
}
```

限制特定用户访问

```nginx

```

基于HTTP响应状态码做权限控制

```nginx
location /private/{
  auth_request /auth;
  index login.html; #如果/auth返回200，则显示login.html
}
location /auth {
  proxy_pass http://127.0.0.1:8080/verify;
  
}
```

`return`

```nginx
location /{
  return 200 "return 200 HTTP Code"; #return code[text]
	return 302 /bbs; #return code URL，重定向
  return http://192.168.184.240:8000/bbs; #return URL，必须为http或https
}
```

`rewrite`

```nginx
#rewrite regex replacement [flag]
#flag可选值
/*
last:重写后url发起新请求，再次进入server段重试location的匹配
break:直接使用重写的url，不再匹配location
redirect:返回302临时重定向
permanent:返回301临时重定向
*/
server{
  listen 80;
  root html;
  location /search{
    rewrite ^/(.*) http://www.baidu.com redirect;
  }
  location /images {
    return /images/(.*) /pics/$1;
  }
  location /pics {  
  }
}
```

`if`语句

```nginx
#condition
/*
$variable 值为空或以0开头字符串当作false
=,!=
~,!~ 正则匹配
~* 正则匹配，不区分大小写
-f !-f 文件是否存在
-d !-d 目录是否存在
-e !-e 文件、目录、符号连接是否存在
-x !-x 文件是否可执行
*/
server {
	listen 8080;
  location /search/ {
    if ($remote_addr = "192.168.184.1") {
    	return 200 "test if OK";
    } 
  }
}
```

`autoindex`

```nginx
#用户请求以/结尾时，列出目录结构
server {
	listen 80;
  location /download/ {
    autoindex on;
    autoindex_exact_size on;
    autoindex_format html; #json,xml格式不可下载
    autoindex_localtime off;
  }
}
```

### 变量

| 变量                | 含义                            |
| ------------------- | ------------------------------- |
| remote_addr         |                                 |
| remote_port         |                                 |
| server_port         |                                 |
| server_addr         |                                 |
| server_protocol     |                                 |
| binary_remote_addr  |                                 |
| connection          | TCP连接序号，递增               |
| connection_request  | TCP连接请求数量                 |
| proxy_protocol_addr |                                 |
| uri                 | 不包含参数                      |
| request_uri         | 包含参数                        |
| scheme              | 协议名，http或https             |
| request_method      | 请求方法                        |
| request_length      | 包括请求行，请求头，请求体      |
| args                | 全部参数字符串                  |
| arg_参数名          | 获取特定参数值                  |
| is_args             | URL中有参数，则返回？否则返回空 |
| remote_user         | HTTP Basic传入的用户名          |
| query_string        | 同args                          |

特殊变量

| host                 | 请求行、头，最后找server_name |
| -------------------- | ----------------------------- |
| http_user_agent      | 用户浏览器                    |
| http_referer         | 从哪些链接过来的请求          |
| http_via             | 经过代理服务器，添加其信息    |
| http_x_forwarded_for | 用户真实IP                    |
| http_cookie          |                               |

处理HTTP请求的变量

| 变量               | 含义                              |
| ------------------ | --------------------------------- |
| request_time       |                                   |
| request_completion | 处理完成返回OK                    |
| server_name        |                                   |
| https              | 开启https则返回on                 |
| request_filename   | 访问文件完整路径                  |
| document_root      | URI和root/alias规则生成的文件路径 |
| realpath_root      | document_root中软链接换成真实路径 |
| limit_rate         |                                   |

### 反向代理

| 指令               | 含义                         |
| ------------------ | ---------------------------- |
| upstream           | 上游服务URL                  |
| server             | 上游服务地址                 |
| zone               | 定义共享内存，跨worker子进程 |
| keepalive          | 对上游服务启用长连接         |
| keepalive_requests | 一个长连接最多请求个数       |
| keepalive_timeout  |                              |

配置后端服务器：192.168.184.20

```nginx
server {
	listen 8080;
	server_name localhost;
  location /proxy {
  	root /opt/nginx/html/app;
    index proxy.html;
  }
}
```

```shell
#!/bin/bash
DIR=/opt/nginx/html/app/proxy
FILE=proxy.html
while true;do
	echo "Application Server, this time create number: $RANDOM" > $DIR/$FILE
	sleep 1
	done
```

```shell
nohup sh create_random_number.sh &
```

配置代理服务器：192.168.184.240

```nginx
upstream back_end{
  server 192.168.184.20:8080 weight=2 max_conns=1000 fail_timeout=10s max_fails=3;
  keepalive 32;
  keepalive_requests 80;
  keepalive_timeout 20s;
}
server {
  listen 80;
  server_name proxy.ecnuonion.com;
  location /proxy {
    proxy_pass http://back_end/proxy; #back_end替换为192.168.184.20:8080
  }
}
```

添加域名解析/etc/hosts

```shell
192.168.184.240 proxy.ecnuonion.com
```

测试

```shell
curl proxy.ecnuonion.com/proxy/
```

注意

proxy_pass不带/意味着nginx不会修改用户URL，带/nginx会将location后的url从用户url中删除

例

```nginx
location /bbs/{
	proxy_pass http://localhost:8080/;
}
#用户请求:/bbs/abc/test.html，到达上游服务器:/abc/test.html
```

请求头

```nginx
server {
	listen 80;
	location /receive/ {
		proxy_pass http://back_end/proxy;
    client_max_body_size 250k;
    client_body_buffer_size 100k;
    client_body_temp_path test_body_path;
    client_body_in_file_only on;
    client_body_in_single_buffer on;
    proxy_request_buffering on;
    client_body_timeout 30;
	}
}
```

上游服务器：192.168.184.20

```nginx
server {
  listen 8888;
  server_name localhost;
  location / {
    return 200 "以下是请求行变量：
      request_method: $request_method
      uri: $uri
      request: $request
      以下是请求头变量：
      http_test: $http_test
      content_length: $content_length
      content_type: $content_type";
  }
}
```

代理服务器：192.168.184.240

```nginx
server {
	listen 8010;
  server_name proxy.ecnuonion.com;
  location /request/ {
    proxy_pass http://192.168.184.20:8888;
  }
}
```

测试

```shell
curl -H "test: 'Test Variable'" proxy.ecnuonion.com:8010/request/
/*
以下是请求行变量：
request_method: GET
uri: /request/
request: GET /request/HTTP/1.0
以下是请求头变量：
http_test: Test Variable
content_length: 
content_type: 
*/
curl -H "test: 'Test Variable'" -d "name=jack&age=25&score=99" proxy.ecnuonion.com:8010/request/
/*
以下是请求行变量：
request_method: POST
uri: /request/
request: POST /request/HTTP/1.0
以下是请求头变量：
http_test: Test Variable
content_length: 25
content_type: application/x-www-form-urlencoded
*/
```

修改代理服务器配置

```nginx
server {
	listen 8010;
  server_name proxy.ecnuonion.com;
  location /request/ {
    proxy_pass http://192.168.184.20:8888;
    proxy_method PUT;
    proxy_http_version 1.1;
    proxy_set_header test "var modify by nginx";
    #proxy_pass_request_headers off;
    #proxt_pass_request_body off;
  }
}
```

测试

```nginx
curl -H "test: 'Test Variable'" proxy.ecnuonion.com:8010/request/
/*
以下是请求行变量：
request_method: PUT
uri: /request/
request: PUT /request/HTTP/1.1
以下是请求头变量：
http_test: var modify by nginx
content_length: 
content_type: 
*/
```

### 负载均衡

```nginx
upstream demo_server {
  #hash $request_uri;
  #ip_hash; 根据remote addr
	server 192.168.184.20:8020 weight 2;
  server 192.168.184.20:8020 weight 1;
}
server {
  lisren 80;
  server_name balance.ecnuonion.com;
  location /balance/{
    proxy_pass http://demo_server;
  }
}
```

```nginx
upstream demo_server {
  zone test 10M; #共享内存
  least_conn; #最少连接数算法
	server 192.168.184.20:8020 ;
  server 192.168.184.20:8020 ;
}
```

异常容错机制

```nginx
server {
  location /test/ {
    proxy_pass http://test_tolerant_server;
    proxy_next_upstream off; #默认为error timeout
  }
}
```

模拟超时

上游服务器

```nginx
server {
  listen 4040;
  location / {
    return 200 "server 4040\n";
  }
}
server {
  listen 4050;
  location / {
    set $limit_rate 1;
    return 200 "server 4050\n";
  }
}
```

代理服务器

```nginx
server {
  location /test/ {
    proxy_pass http://test_tolerant_server;
    proxy_next_upstream timeout; #默认为error timeout
    proxy_read_timeout 5;
  }
}
```

`proxy_intercept_errors`

```nginx
#上游返回响应码大于300时，是直接将上游响应返回客户端还是按照error_page处理
server {
  root /opt/nginx/html;
  location /503.html { 
  }
  location /test/ {
    proxy_pass http://test_tolerant_server;
    error_page 503 /503.html;
    proxy_intercept_errors on; #默认为off
  }
}
```

### 缓存

### HTTPS

信息加密+完整性校验+身份验证

连接建立阶段使用非对称加密

数据传输阶段使用对称加密

#### 配置CA服务器(120.77.176.55)

```shell
cd /etc/pki/CA/
(umask 077;openssl genrsa -out private/cakey.pem 1024)
openssl req -new -x509 -key private/cakey.pem -out cacert.pem -days 365
touch index.txt serial
echo "10001" > serial
mkdir csr
```

#### 组织机构向CA申请证书(139.9.217.28)

```shell
mkdir /opt/nginx/https -pv
(umask 077;openssl genrsa -out ecnuonion.edu.key 1024)
openssl req -new -key ecnuonion.edu.key -out ecnuonion.edu.csr
scp ecnuonion.edu.csr root@120.77.176.55:/etc/pki/CA/csr
```

#### CA签发证书

```shell
cd /etc/pki/CA/
openssl ca -in csr/ecnuonion.edu.csr -out ecnuonion.edu.crt -days 365
scp ecnuonion.edu.crt root@120.77.176.55:/etc/pki/CA/csr
```



