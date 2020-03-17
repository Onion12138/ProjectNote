## Linux

### 网络

#### 网络配置文件

```shell
vim /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0
BOOTPROTO=none #是否自动获取IP(none，static，dhcp)，局域网内必须有dhcp服务器才可以设置为dhcp
HWADDR=00:0c:29:17:c4:09 #MAC地址
ONBOOT=yes #是否随网络服务启动，eth0生效
TYPE=Ethernet
UUID=...
IPADDR=...
NETMASK=...
GATEWAY=...
IPV6INIT=noi
USERCTL=no #不允许非root账户控制此网络
```



```shell
vim /etc/sysconfig/network

NETWORKING=yes
HOSTNAME=localhost.localdomain
```

DNS配置文件

```shell
vim /etc/resolv.conf
```

#### 常见命令

- ifconfig 查看与配置网络状态命令

```shell
ifconfig eth0 192.168.0.200 netmask 255.255.255.0 #临时设置
ifconfig eth0 down
```

- netstat

-t 列出TCP协议端口

-u 列出UDP协议端口

-n 使用IP地址和端口号

-l 仅列出在监听状态网络服务

-a 列出所有网络连接

netstat -rn 列出路由列表，功能和route -n一致

```shell
netstat -an | grep ESTABLISHED | wc -l
```

- route

route -n 查看路由列表

- nslookup DNS解析



### ssh

#### 配置文件

`vim /etc/ssh/sshd_config`

```shell
port 22
ListenAddress 0.0.0.0
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
GSSAPIAuthentication yes #建议客户端改为no
PermitRootLogin no #是否允许root的ssh登录，建议为no，默认为yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication yes
```

#### sftp命令

sftp root@120.77.176.55

ls 服务端目录

lls 客户端目录

lcd 切换本地目录

get 下载

put 上传

#### 密钥对验证

1. 客户端创建密钥对

id_rsa

id_rsa.pub

ssh-keygen -t rsa

2. 上传公钥id_rsa.pub
3. 导入到服务端用户onion的公钥数据库~/.ssh/authorized_keys
4. 以服务端用户onion的身份登录

client端

```shell
ssh-keygen -t rsa
scp id_rsa.pub root@192.168.1.10:/root
```

server端

```shell
cat id_rsa.pub >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys

vim /etc/ssh/sshd_config

PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
PasswordAuthentication no

service sshd restart

```









