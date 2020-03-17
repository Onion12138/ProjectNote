

# SpringBoot开发手册

## 第〇章： 规范
## 第一章： 基础配置
### 日志
### 配置文件

- 属性注入

1. @value

注意：变量不可以为static类型

```Java
@Value("${qiniu.access-key}")//${}:从配置文件中读取
private String accessKey;
@Value("#{10*2}") //#{}Spring表达式
private Integer age;
```

2. @ConfigurationProperties(prefix = "...")

要求

(1)组件必须在ioc容器中

(2)和yml配置文件字段一致

## 第二章： Spring特性
IOC

AOP：面向切面编程

一、切面表达式

within

```java
@Pointcut("within(com.imooc.service.ProductService)") //ProductService类所有方法
@Pointcut("within(com.imooc..v//包及子包下所有类的方法
```

匹配对象

```java
//DemoDao实现IDao接口
@Pointcut("this(com.imooc.DemoDao)") //匹配AOP对象的目标对象为指定类型的方法，即DemoDao的AOP代理对象的方法
@Pointcut("target(com.imooc.IDao)") //匹配实现IDao接口的目标对象，这里即DemoDao的方法
@Pointcut("bean(*Service)") //匹配所有以Service结尾的bean里面的方法
```

匹配参数

```java
@Pointcut("execution(* *..find*(Long))")//匹配任何以find开头且只有一个Long参数的方法
@Pointcut("execution(* *..find*(Long,..))")//匹配任何以find开头且第一个参数为Long的方法
@Pointcut("args(Long)") //匹配任何只有一个Long参数的方法
@Pointcut("args(Long,..)")//匹配第一个参数为Long的方法
```

匹配注解

```java
@Pointcut("@annotation(com.imooc.annotation.AdminOnly)")//方法标注有AdminOnly注解
@Pointcut("@within(com.google.common.annotations.Beta)")//annotation的RetentionPolicy为class，标注有Beta的类下的方法
@Pointcut("@target(org.springframework.stereotype.Repository)")//annotation的RetentionPolicy为runtime，标注有Repository的类下的方法
@Pointcut("@args(org.springframework.stereotype.Repository)") //传入的参数类标柱有Repository注解的方法
```

execution()表达式

访问修饰符 返回值 包名.包名.包名...类名.方法名(参数列表)

1. 标准的表达式写法：

public void com.itheima.service.impl.AccountServiceImpl.saveAccount()

2. 访问修饰符可以省略

void com.itheima.service.impl.AccountServiceImpl.saveAccount()

3. 返回值可以使用通配符，表示任意返回值

\* com.itheima.service.impl.AccountServiceImpl.saveAccount()

4. 包名可以使用通配符，表示任意包。但是有几级包，就需要写几个*.

 \* \*.\*.\*.\*.AccountServiceImpl.saveAccount())

5. 包名可以使用..表示当前包及其子包

\* *..AccountServiceImpl.saveAccount()

6. 类名和方法名都可以使用*来实现通配

\* \*..\*.\*()

 参数列表：

1. 可以直接写数据类型

   基本类型直接写名称      int

   引用类型写包名.类名的方式  java.lang.String

2. 可以使用通配符表示任意类型，但是必须有参数
3. 可以使用..表示有无参数均可，有参数可以是任意类型

全通配写法：

​          \* \*..\*.\*(..)

 实际开发中切入点表达式的通常写法：

切到业务层实现类下的所有方法

 \* com.itheima.service.impl.\*.\*(..)

二、Advice注解

@Before

```Java
@Before("match() && args(productId)")
public void before(Long productId){
  	System.out.println("Id : " + productId);
}
//获取方法名，参数
@Before("")
public void before(JoinPoint joinPoint){
    String methodName = joinPoint.getSignature().getName();
    List<Object> args  = Arrays.asList(joinPoint.getArgs());
}
```

@After

@AfterReturning

可以获取方法返回值

```java
@AfterReturning(value = "matchReturn", returning = "result")
public void after(Object result){
  	System.out.println("result : " + result);
}
```

@AfterThrowing

```Java
@AfterReturning(value = "matchThrow", throwing = "ex")
public void after(Exception ex){
  	ex.printStackTrace();
}
```

@Aroud 

在目标方法执行前后都执行

```Java
@Around("verify()")
public Object verifyLogin(ProceedingJoinPoint pjp) throws Throwable{
		Object result;
  	try{
      	System.out.println("前置通知");
    		result = pjp.proceed(pjp.getArgs());
      	System.out.println("返回通知");
  	}catch(Throwable e){
    		System.out.println("异常通知");
  	}
  	System.out.println("后置通知");
  	return result;
}
```

三、优先级

使用@Order指定切面优先级，值越小，优先级越高

## 第三章： Web

> 参数绑定

自定义转换器

```Java
@Configuration
public class DateConfig {
    /** 默认日期时间格式 */
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    /** 默认日期格式 */
    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    /** 默认时间格式 */
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";
    /**
     * LocalDate转换器，用于转换RequestParam和PathVariable参数
     */
    @Bean
    public Converter<String, LocalDate> localDateConverter() {
        return new Converter<>() {
            @Override
            public LocalDate convert(String source) {
                return LocalDate.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT));
            }
        };
    }
    /**
     * LocalDateTime转换器，用于转换RequestParam和PathVariable参数
     */
    @Bean
    public Converter<String, LocalDateTime> localDateTimeConverter() {
        return new Converter<>() {
            @Override
            public LocalDateTime convert(String source) {
                return LocalDateTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT));
            }
        };
    }
    /**
     * LocalTime转换器，用于转换RequestParam和PathVariable参数
     */
    @Bean
    public Converter<String, LocalTime> localTimeConverter() {
        return new Converter<>() {
            @Override
            public LocalTime convert(String source) {
                return LocalTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT));
            }
        };
    }
    /**
     * Date转换器，用于转换RequestParam和PathVariable参数
     */
    @Bean
    public Converter<String, Date> dateConverter() {
        return new Converter<>() {
            @Override
            public Date convert(String source) {
                SimpleDateFormat format = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                try {
                    return format.parse(source);
                } catch (ParseException e) {
                    throw new RuntimeException(e);
                }
            }
        };
    }

    /**
     * Json序列化和反序列化转换器，用于转换Post请求体中的json以及将我们的对象序列化为返回响应的json
     */
    @Bean
    public ObjectMapper objectMapper(){
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        objectMapper.disable(DeserializationFeature.ADJUST_DATES_TO_CONTEXT_TIME_ZONE);

        //LocalDateTime系列序列化和反序列化模块，继承自jsr310，我们在这里修改了日期格式
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addSerializer(LocalDateTime.class,new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addSerializer(LocalDate.class,new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addSerializer(LocalTime.class,new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDateTime.class,new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDate.class,new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addDeserializer(LocalTime.class,new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        //Date序列化和反序列化
        javaTimeModule.addSerializer(Date.class, new JsonSerializer<>() {
            @Override
            public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                SimpleDateFormat formatter = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String formattedDate = formatter.format(date);
                jsonGenerator.writeString(formattedDate);
            }
        });
        javaTimeModule.addDeserializer(Date.class, new JsonDeserializer<>() {
            @Override
            public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {
                SimpleDateFormat format = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String date = jsonParser.getText();
                try {
                    return format.parse(date);
                } catch (ParseException e) {
                    throw new RuntimeException(e);
                }
            }
        });
        objectMapper.registerModule(javaTimeModule);
        return objectMapper;
    }
}
```

> 统一异常处理与返回格式

1. 统一异常

```Java
//自定义枚举类
@Getter
public class ResultEnum {
    ;
    private int code;
    private String message;
    ResultEnum(int code, String message){
        this.code = code;
        this.message = message;
    }
}
//自定义异常
@Getter
public class MyException extends RuntimeException{
    private Integer code;
    public MyException(ResultEnum resultEnums){
        super(resultEnums.getMessage());
        this.code=resultEnums.getCode();
    }
    public MyException(String message, Integer code){
        super(message);
        this.code=code;
    }
}
```

2. 统一返回格式

```java
@Data
public class ResultEntity {
    private Object data;
    private Integer code;
    private String message;
    private ResultEntity(Object data, Integer code, String message){
        this.data = data;
        this.code = code;
        this.message = message;
    }
    public static ResultEntity succeed(){
        return new ResultEntity(null, 0, "成功");
    }
    public static ResultEntity succeed(Object data){
        return new ResultEntity(data, 0, "成功");
    }
    public static ResultEntity fail(ResultEnum resultEnum){
        return new ResultEntity(null, resultEnum.getCode(), resultEnum.getMessage());
    }
    public static ResultEntity fail(String message){
        return new ResultEntity(null, -1, message);
    }
}
```

3. 统一异常处理切面

```Java
@RestControllerAdvice
public class MyExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    public ResultEntity exception(Exception e){
        e.printStackTrace();
        return ResultEntity.fail(e.getMessage());
    }
}
```

> 文件上传

- 基于SpringMVC
- 基于七牛云平台

> Request

获取HttpServletRequest

```Java
HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
HttpSession session = request.getSession();
```

>  Response

> 数据校验

pom文件

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
 </dependency>
```

常用注解如下

日期类

@Future //只能为将来时期
@Past
@DateTimeFormat(pattern = "yyyy-MM-dd")

字符串类

@NotNull(message = "...")

@NotEmpty

@Size(min = 6, max = 16) //字符串

数值类

@DecimalMin(value = "0.1")
@Min(value = 1)
@Range(min = 1, max = 888) //整型

其他

@Email

被校验对象加@Valid或@Validated，两个注解的区别请上网查阅

自定义校验规则

```java
public class MyValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        return clazz.equals(UserRequest.class); //需要验证的POJO类型
    }
    @Override
    public void validate(Object target, Errors errors) {
        if (target == null){
            errors.rejectValue("",null, "用户不能为空");
        }
        UserRequest user = (UserRequest) target;
        if (StringUtils.isEmpty(user.getAccount())){
            errors.rejectValue("account", null, "用户Id不能为空");
        }
    }
}
//还需要在Controller中添加InitBinder
@InitBinder
public void initBinder(WebDataBinder binder){
     binder.setValidator(new MyValidator());
}
```

使用AOP校验参数

```java
@Before("verifyParamsPoint()")
public void verifyParams(JoinPoint joinPoint){
     Object[] args = joinPoint.getArgs();
     BindingResult result = (BindingResult) args[1]; //规定第二个参数为BindingResult
     if (result.hasErrors()){
         StringBuilder sb = new StringBuilder();
         result.getAllErrors().forEach(e->{
             FieldError error = (FieldError) e;
             String field = error.getField();
             String message = error.getDefaultMessage();
             sb.append(field).append(" : ").append(message).append(' ');
         });
         throw new MyException(sb.toString(), -1);
    }
}
```

## 第四章： 数据库
- Jpa

> JPQL查询

更新删除操作，需要手动添加事务支持。默认执行结束后回滚事务，设置@Rollback(value=false)

```java
@Query(value = "from user where nickname = ?")
    User findUser(String nickname);
@Query(value = "update User set nickname = ?2 where id = ?1")
@Modifying
    void updateUser(String id, String nickname);
```

> SQL查询

```java
@Query(value = "select * from user where nickname = ?" ,nativeQuery = true)
    User findUser(String nickname);
```

> 方法命名查询



> 动态查询

继承

```java
interface UserDao extends JpaRepository<User,Long>, JpaSpecificationExecutor<User>{
}
```

示例：and

```java
@Test
public void test1(){
    Specification<User> specification = new Specification<User>() {
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                Path<Object> nickname = root.get("nickname");
                Path<Object> email = root.get("email");
                Predicate p1 = criteriaBuilder.equal(nickname, "onion");
                Predicate p2 = criteriaBuilder.equal(email, "969023014@qq.com");
                return criteriaBuilder.and(p1, p2);
            }
        };
    userDao.findOne(specification);
}
```

示例：模糊查询

```java
@Test
public void test2(){
     Specification<User> specification = (Specification<User>) (root, criteriaQuery, criteriaBuilder) -> {
            Path<Object> nickname = root.get("nickname");
            return criteriaBuilder.like(nickname.as(String.class),"onion%");
     };
     userDao.findOne(specification);
}
```

> 多表操作

一、一对多

一个公司由多个雇员，一个雇员属于一个公司

主表

```java
@OneToMay(targetEntity = Employee.class)
@JoinColumn(name = "employee_company_id", referencedColumnName = "company_id")
//name：外键名 referencedColumnName：参照的主表的主键名
private set<Employee> employees
```

从表

```Java
@ManyToOne(targetEntity = Company.class)
@JoinColumn(name = "employee_company_id", referencedColumnName = "company_id")
private Company company;
```

添加双向联系

```java
employee.setCompany(company); //对外键赋值
company.getEmployees.add(employee); 
```

放弃外键维护

```java
@OneToMay(mappedBy = "company") //参照Employee中的company
private set<Employee> employees;
```

级联操作

```java
@OneToMay(mappedBy = "company", cascade = CascadeType.ALL)
private set<Employee> employees;
```

对象导航查询

从一查多，调用get方法，默认延迟加载，调用get方法并不会立即查询，而在使用时进行查询

可以修改配置

```java
@OneToMay(mappedBy = "company", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
```

从多查一，默认为立即加载

二、多对多

一个用户对应多个角色，一个角色对应多个用户

```Java
@ManyToMany(targetEntity = Role.class)
@JoinTable(name = "user_role",
joinColumns = {@JoinColumn(name = "sys_user_id", referencedColumnName = "user_id")},
inverseJoinColumns = {@JoinColumn(name = "sys_role_id", referencedColumnName = "role_id")})
private Set<Role> roles;
```

被动的一方放弃维护权

```java
@ManyToMany(mappedBy = "roles")
private Set<User> users;
```

```Java
user.getRoles().add(role);
role.getUsers().add(user);
```

- redis

> 基本命令

```bash
#client端
clear
help 命令
help @string
quit/exit
```

`string`

```bash
set key value
get key
del key #成功返回integer 1，失败返回integer 0

mset key1 value1 key2 value2
mget key1 key2 
strlen key
append key value #存在追加，不存在新建，返回追加后的长度

incr key
incrby key increment #上限：Long.MAX_VALUE
incrbyfloat key increment
decr key
decrby key increment

setex key seconds value #如果又进行了set操作，setex操作会失效
psetex key milliseconds value
```

`hash`

```bash
hset key field value
hget key filed
hgetall key
hdel key field1 [field2]
hmset key field1 value1 field2 value2
hmget key field1 field2
hlen key
hexists key field
hkeys key #获取所有fields
hvals key #可能会重复
hincrby key field increment
hincrbyfloat key field increment
hsetnx key field value #不存在才添加
#案例：购物车
```

`list`双向链表

```bash
lpush key value1 [value2] 
rpush key value1 [value2]
lrange key start stop #start从0开始，闭区间。查询全部:lrange key 0 -1
lindex key index
llen key
lpop key #查看并移除
rpop key
blpop key1 [key2] timeout #阻塞，规定时间内获取数据
brpop key1 [key2] timeout
lrem key count value #用于具有操作先后顺序的数据控制，案例：微信朋友圈点赞按时间顺序排列
```

`set`

```bash
sadd key member1 [member2]
smembers key
srem key member1 [member2]
scard key #返回数据总量，案例：统计网站访问量，需要去重
sismember key member
srandmember key [count]
spop key #随机获取某个数据并将其移出集合
sinter key1 [key2]
sunion key1 [key2]
sdiff key1 [key2]
sinterstore destination key1 [key2]
sunionstore destination key1 [key2]
sdiffstore destination key1 [key2]
smove source destination member
```

`sorted_set`

```bash
zadd key score1 member1
zrange key start stop [withscores]
zrevrange key start stop #反向
zrem key member
zrangebyscore key min max
zrevrangebyscore key max min
zremrangebyrank key start stop
zremrangebyscore key min max
zcard key
zcount key min max
zinterstore destination numkeys key
zunionstore destination numkeys key
zrank key member #查索引，排名
zrevrank key member
zscore key member #案例：排行榜
zincrby key increment member
```

`key`的相关命令

```shell
del key
exists key
type key

expire key seconds
pexpire key milliseconds
expireat key timestamp
pexpireat key milliseconds-timestamp

ttl key #查看有效期，不存在返回-2，存在返回-1，如果设置了有效期返回剩余有效期
pttl key

persist key #转换为永久性

keys pattern #keys * 查看所有 *匹配任意数量字符，？匹配一个字符，[]匹配指定符号

rename key newkey
renamenx key newkey #newkey不存在才能修改

sort

```

数据库的通用指令

```
select index
move key db

dbsize
flushdb
flushall
```

>  持久化

- RDB方式

1. 命令行输入 save指令，生成dump.rdb

修改配置

```
dbfilename dump-6379.rdb
rdbcompression yes
rdbchecksum yes
```

save指定会阻塞Redis服务器

2. 命令行输入bgsave指令

后台执行，子进程完成

3. 配置

```shell
save 900 1 #900秒1个变化，则持久化
save 300 10
save 60 10000
```

| 方式           | save | bgsave |
| -------------- | ---- | ------ |
| 读写           | 同步 | 异步   |
| 阻塞客户端指令 | 是   | 否     |
| 额外内存消耗   | 否   | 是     |
| 启动新进程     | 否   | 是     |

- AOF

记录数据产生的过程

三种写策略

​	always 数据零误差，性能低

​	everysec 系统突然宕机，丢失1秒数据

​	no 系统控制

开启配置

```shell
appendonly yes
appendfsync always
appendfilename filename #自定义AOF文件名
dir #AOF文件持久化保存路径
```

AOF重写

指定：bgrewriteaof

配置

```shell
auto-aof-rewrite-min-size size
auto-aof-rewrite-percentage percentage
```

触发条件

```shell
aof_current_size > auto-aof-rewrite-min-size
aof_current_size - aof_base_size / aof_base_size >= auto-aof-rewrite-percentage
```

查看：输入info指令查看aof_current_size和aof_base_size

| 指标         | RDB              | AOF              |
| ------------ | ---------------- | ---------------- |
| 占用存储空间 | 小(数据级，压缩) | 大(指令级，重写) |
| 存储速度     | 慢               | 快               |
| 回复速度     | 快               | 慢               |
| 数据安全性   | 会丢失           | 根据策略而定     |
| 资源消耗     | 高               | 低               |
| 启动优先级   | 低               | 高               |

> 事务

开启：multi

放弃：discard

执行：exec

如果命令格式错误，则事务中所有命令均不执行

如果命令格式正确，执行时出错，则仅有错误命令不会执行



锁：

 watch key 在multi之前

unwatch 

如果在事务中key被修改，则exec会返回nil，事务终止



分布式锁：

setnx lock-key value

设置失效：expire lock-key 20

删除：del lock-key

> 删除策略

```shell
maxmemory-policy no-enviction #4.0默认，引发OOM
/*
针对server.db[i].expires
volatile-lru
volatile-lfu
volatile-ttl
volatile-random
针对server.db[i].dic
allkeys-lru
allkeys-lfu
allkeys-random
*/
```

> 配置文件

```shell
daemonize yes|no
bind 127.0.0.1
databases 16
loglevel debug|verbose|notice|warning #默认verbose
logfile 端口号.log
maxclients 0
timeout 300 #300s
include /path/server-端口.conf

```

> 高级数据结构

`bitmaps`

```shell
setbit bits 0 1
getbit bits 0 #如果不存在，返回0

bitop op destKey key1 [key2]
/* op操作
and
or
not
xor
*/
bitcount key [start end] #统计key中1的数量

#案例:电影点播
setbit 20880808 0 1
setbit 20880808 4 1
setbit 20880808 8 1
setbit 20880809 0 1
setbit 20880809 5 1
bitcount 20880808 #3
bitcount 20880809 #2
bitop or 08-09 20880808 20880809
```

`HyperLogLog`

不保存数据，只保存数量

基数估算算法，最终数值存在一定误差

存储空间消耗极小，12K内存

```
pfadd key element 
pfcount key
pfmerge destkey sourcekey
```

`GEO`

```shell
geoadd key longitude latitude member
geopos key member
geodist key member1 member2 [unit] #m,km
georadius key longitude latitude radius m|km
georadiusbymember key member radius m|km
```

> 主从复制

master：6379 slave：6380

```shell
#slave客户端发送命令
slaveof 127.0.0.1 6379
#master服务端启动
redis-server --slaveof 127.0.0.1 6379
#slave服务端配置文件
slaveof 127.0.0.1 6379

#slave客户端断开连接
slaveof no one
```

授权访问

```shell
#master配置文件
requirepass <password>
#master客户端发送命令
config set requirepass <password>
config get requirepass
#slave客户端发送命令设置密码
auth <password>
#slave配置文件设置
masterauth <password>
#启动客户端设置
redis-cli -a <password>
```

> 哨兵

```shell
#启动
redis-sentinel sentinel-端口.conf
#配置
port 26379
dir /redis-4.0.0/data
sentinel monitor mymaster 127.0.0.1 6379 2 #多少个哨兵认定master宕机
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

> 集群

```shell
#配置
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 10000
#查看版本
ruby -v
gem -v
#输入命令
./redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384
#3主:6379 6380 6381
#3从:6382 6383 6384

#集群下设置与获取数据，客户端启动
redis-cli -c
```

- Mongodb

> 数据库逻辑结构

| Mongodb     | Mysql     |
| ----------- | --------- |
| databases   | databases |
| collections | table     |
| document    | row       |

> 数据类型

| 类型   | 示例             | 备注                                                   |
| ------ | ---------------- | ------------------------------------------------------ |
| null   | {"x":null}       |                                                        |
| bool   | {"x":true}       |                                                        |
| 数值   | {"x":3.14}       | 默认64位浮点，可指定{"x":NemberInt("3")}，NemberLong同 |
| 字符串 | {"x":"呵呵"}     |                                                        |
| 二进制 |                  | 将非utf字符保存到数据库唯一方式                        |
| 日期   | {"x":new Date()} | 自新纪元经过的毫秒数，不存储时区                       |
| 正则   | {"x":/[abc]/}    | 语法同javascript，查询时使用正则作为限定条件           |
| 数组   | {"x":["a","b"]}  |                                                        |
| 文档   | {"x":{"y":3}}    |                                                        |
| Id     | {"x":objectId()} | 12字节字符串，文档唯一标识                             |
| 代码   |                  | 任何Javascript代码                                     |

> 增删改查

- 选择和创建

use 数据库名

- 插入

```
db.spit.insert({content:"this is content",userid:"1011",visits:NumberInt(902)})
```

- 查询

```
db.spit.find({userid: '1013'})
db.spit.findOne({userid: '1013'})
db.spit.find().limit(3)
db.spit.find({content:/流量/}) //模糊匹配
db.spit.find({content:/^流量/}) //匹配开头
db.spit.find({visits:{$gt:1000}}) //大于1000，其他的还有gt,lt,gte,lte,ne
db.spit.find({userid:{$in:["1013","1014"]}}) //不在：nin
db.spit.find({$and:[{visits:{$gte:1000}},{visits:{$lt:2000}}]}) //$and:[{}.{}]


db.account.find({contact:{$all:["China","Beijing"]}}) //满足所有
db.account.find({contact:{$all:[["2222","3333"]]}})
db.account.find({contact:{$elemMatch:{$gt:"1000",$lt:"2000"}}}) //满足一个

db.account.find({name:{$in:[/^c/,/^j/]}}) 
db.account.find({name:{$regex: /LIE/,$options:'i'}}) //名字包含LIE，不区分大小写

db.account.find({},{name:1}) //投影，1表示返回该字段。默认返回文档主键
db.account.find({},{name:1,_id:0}) //不返回文档主键，不可以混用包含和不包含，除了ID
db.account.find({},{_id:0,name:1,contact:{$slide:1}}) //数组，返回第一个元素。设置-1，返回最后一个元素
db.account.find({},{_id:0,name:1,contact:{$slide:[1,2]}}) //跳过第一个，返回后面两个元素
db.account.find({},{_id:0,name:1,contact:{$elemMatch:{$gt:"Alabama"}}}) //返回满足筛选条件的第一个元素
db.account.find({contact:{$gt:"Alabama"}},{_id:0,name:1,"contact.$":1}) //同上
```

- 修改

```
db.spit.update({_id:"1"},{visits:NumberInt(1000)})//其他字段会消失
db.spit.update({_id:"1"},{$set:{visits:NumberInt(1000)}}) //部分修改
db.spit.update({_id:"2"},{$inc:{visits:NumberInt(1)}}) //列值增长

db.account.update(
	{name: "jack"},
	{$unset:
		{
			"contact.0": ""
		}
	}
) //删除第一个元素

db.account.update(
{name: "karen"},
{$addToset: {contact:"China"}}
)

db.account.update(
{name: "karen"},
{$addToSet: {contact:{$each:["contact1","contact2"]}}}
)

db.account.update(
	{name:"karen"},
	{$pop:{"contact":-1}} //删除第一个元素
)

```

```
db.account.insert([
	{
		name: "jack",
		balance: 2000,
		contact: ["1111","Alabama","US"]
	},
	{
		name: "karen",
		balance: 2500,
		contact: [["2222","3333"],"Beijing","China"]
	}
])

```



- 删除

```
db.spit.remove({})
db.spit.remove({visits:1000})
```

- 统计

```
db.spit.count()
db.spit.count({userid:"1013"})
```



> 管道聚合

```
db.accounts.insertMany([
	{
		name: {firstName: "alice", lastName: "wong"},
		balance: 50
	},
	{
		name: {firstName: "bob", lastName: "yang"},
		balance: 20
	}
])
```

project

```
db.accounts.aggredate([
	{
		$project: {
			_id: 0, //不显示id
			balance: 1,
			clientName: "$.name.firstName"
		}
	}
])

{"balance": 50, "clientName": "alice"}
{"balance": 20, "clientName": "bob"}

db.accounts.aggredate([
	{
		$project: {
			_id: 0,
			balance: 1,
			nameArray: ["$name.firstName", "$name.middleName", "$name.lastName"]
		}
	}
])

{"balance": 50, "nameArray": ["alice", null, "wong"]}
{"balance": 20, "nameArray": ["bob", null, "yang"]}
```

match

```
db.accounts.aggredate([
	{
		$match: {
			"name.firstName": "alice"
		}
	}
])

db.accounts.aggredate([
	{
		$match: {
			$or: [
				{ balance: { $gt: 40, $lt: 80}},
				{"name.lastName": "yang"}
			]
		}
	},
	{
		$project: {
			_id: 0
		}
	}
])

```

limit&skip

```
db.accounts.aggredate([
	{ $limit: 1} // 同样的用法：{$skip: 1} 
])
```

unwind

```
db.accounts.update(
	{"name.firstName": "alice"},
	{$set: {currency: ["CNY","USD"]}}
)
db.accounts.update(
	{"name.firstName": "bob"},
	{$set: {currency: "GBP"}}
)

db.accounts.aggregate([
	{
		$unwind: {
			path: "$currency"
		}
	}
])

{"_id": ObjectId("...b"), "name": {"firstName": "alice", "lastName": "wong"}, "balance": 50, "currency": "CNY"}
{"_id": ObjectId("...b"), "name": {"firstName": "alice", "lastName": "wong"}, "balance": 50, "currency": "USD"}
{"_id": ObjectId("...c"), "name": {"firstName": "bob", "lastName": "yang"}, "balance": 20, "currency": "GBP"}

db.accounts.aggregate([
	{
		$unwind: {
			path: "$currency",
			includeArrayIndex: "ccyIndex"
		}
	}
])

{"_id": ObjectId("...b"), "name": {"firstName": "alice", "lastName": "wong"}, "balance": 50, "currency": "CNY", "ccyIndex": NumberLong(0)}
{"_id": ObjectId("...b"), "name": {"firstName": "alice", "lastName": "wong"}, "balance": 50, "currency": "USD", "ccyIndex": NumberLong(1)}
{"_id": ObjectId("...c"), "name": {"firstName": "bob", "lastName": "yang"}, "balance": 20, "currency": "GBP", "ccyIndex": null}

```

sort

```
db.accounts.aggregate([
	{
		$sort: {balance: 1, "name.lastName": -1}
	}
])
```

lookup

```
db.forex.insertMany([
	{
		ccy: "USD",
		rate: 6.91,
		date: new Date("2018-12-21")
	},
	{
		"ccy": "GBP",
		"rate": 8.72,
		date: new Date("2018-8-21")
	},
	{
		"ccy": "CNY",
		"rate": 1.0,
		date: new Date("2018-12-21")
	}
])

db.accounts.aggregate([
	{
		$lookup: {
			from: "forex",
			localField: "currency",
			foreignFiled: "ccy",
			as: "forexData"
		}
	}
])

db.accounts.aggregate([
	{
		$unwind: {
			path: "$currency"
		}
	},
	{
		$lookup: {
			from: "forex",
			localField: "currency",
			foreignFiled: "ccy",
			as: "forexData"
		}
	}
])
//不相关查询，查询条件和管道文档没有直接联系
db.accounts.aggregate([
	{
		$lookup: {
			from: "forex",
			pipeline: [
				{ $match:
					{
						date: new Date("2018-12-21")
					}
				}
			],
			as: "forexData"
		}
	}
])
//将特定日期外汇汇率写入余额大于100的银行账户文档
db.accounts.aggregate([
	{
		$lookup: {
			from: "forex",
			let: { bal: "$balance"},
			pipeline: [
				{ $match:
					{ $expr:
						{ $and:
							[
								{$eq: ["$date", new Date("2018-12-21")]},
								{$gt: ["$$bal", 100]}
							]
						}
					}
				}
			],
			as: "forexData"
		}
	}
])

```

group

```
db.transactions.insertMany([
	{
		symbol: "600519",
		qty: 100,
		price: 567.4,
		currency: "CNY"
	},
	{
		symbol: "AMZN",
		qty: 1,
		price: 1377.5
		currency: "USD"
	},
	{
		symbol: "AAPL",
		qty: 2,
		price: 150.7
		currency: "USD"
	}
])
//按照交易货币分组交易记录
db.accounts.aggregate([
	{
		$group: {
			_id: "$currency"
		}
	}
])

{ "_id": "USD"}
{ "_id": "CNY"}

db.accounts.aggregate([
	{
		$group: {
			_id: "$currency",
			totalQty: {$sum: "$qty"},
			totalNotional: {$sum: {$multiply: ["$price", "$qty"]}},
			avgPrice: {$avg: "$price"},
			count: {$sum: 1},
			maxNotional: {$max: {$multiply: ["$price", "$qty"]}}
		}
	}
])
//计算所有文档聚合值
db.accounts.aggregate([
	{
		$group: {
			_id: null,
			totalQty: {$sum: "$qty"},
			totalNotional: {$sum: {$multiply: ["$price", "$qty"]}},
			avgPrice: {$avg: "$price"},
			count: {$sum: 1},
			maxNotional: {$max: {$multiply: ["$price", "$qty"]}}
		}
	}
])
```

out

```
//写出新的集合。如果写出到已经存在的集合，会覆盖之前的文档
db.accounts.aggregate([
	{
		$group: {
			_id: "$currency",
			symbols: {$push: "$symbol"}
		}
	},
	{
		$out: "output"
	}
])

```

> 索引

创建

```
//创建单键索引
db.account.createIndex({
		name: 1 //1表示从小到大排序
})
//查看集合中存在的索引
db.accnout.getIndexes()
//创建复合键索引
db.account.createIndex({
		name: 1,
		balance: -1
})
//创建多键索引：数组字段
db.account.createIndex({
		currency: 1
})
```

查看

```
db.account.explain().find({name: "alice"},{_id: 0, name: 1})
```

删除

```
dp.account.dropIndex({"name": 1, "balance": -1})
```

创建唯一索引

```
db.account.createIndex({name: 1},{unique: true})
```

创建稀疏索引，只将包含索引字段的文档加入到索引中

```
db.account.createIndex({name: 1},{sparse: true})
```

创建有生存时期的索引，只用于单键

```
db.account.createIndex({lastAccess: 1},{expireAfterSeconds: 20})
```

> 整合SpringData

```xml
<dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```



```yaml
spring:
	data:
		mongodb:
			host:
			database:
```



更新点赞数技巧

```java
@Autowired
private MongoTemplate mongoTemplate;
public void updateThumbup(String id){
  	Query query = new Query();
  	query.addCriteria(Criteria.where("_id").is(id));
  	Update update = new Update();
  	update.inc("thumbup",1);
  	mongoTemplate.updateFirst(query,update,"spit");
}
```



- ElasticSearch

> 文档

type名约定用_doc

> API

create

```json
POST test/_doc //自动生成id
```

```json
PUT test/_doc/1?op_type=create //指定id，如果id存在，会报错
```

```json
POST test/_create/1  //若存在，则失败
```

read

```json
GET test/_doc/1
```

```json
GET test/_search 查询全部
```

index

```json
PUT test/_doc/1 //文档存在，版本号加1,删除再写入;id不存在，创建新的文档
```

update

```json
POST test/_update/1 //原文档上更新，指定字段修改或追加，版本号加1
```

delete

```json
DELETE test/_doc/1
```

bulk

```json
POST _bulk
```

单条操作失败并不影响其他操作

mget

```json
GET /_mget
{
		"docs": [
			{
				"_index": "test",
				"_id": "1"
			},
			{
				"_index": "test",
				"_id": "2"
			}
		]
}
```

> 分析

Analyzer API

```json
GET /_analyze
{
  "analyzer": "standard",
  "text": "you Know, for Search"
}
```

```json
POST test/_analyze
{
	"field": "title",
	"text": "you Know, for Search"
}
```

```java
POST /_analyze
{
	"tokenizer": "standard",
  "filter": ["lowercase"],
	"text": "you Know, for Search"
}
```

> 分词器

Standard

- 默认分词器
- 按词切分
- 小写处理

simple

- 非字母去除
- 小写处理
- 分割线单词被切为两个

whitespace

- 空格切分
- 大小写保留

stop

- 去掉停用词
- 其他同Simple

keyword

- 不分词

pattern

- 正则表达式分词
- 小写处理

english

- 去时态、复数

> 搜索

基于term的查询

- 对输入不分词，利用相关度算分公式为每个包含该词项的文档进行相关度算分
- 可以通过Constant Score将查询转化为一个Filter避免算分

```json
POST test/_search
{
  "query":{
    "term":{
      "field1":{
        "value": "..."
      }
    }
  }
}
```

```json
POST test/_search
{
  "query":{
    "term":{
      "field1.keyword":{
        "value": "..."
      }
    }
  }
}
```

```json
POST test/_search
{
	"explain": true,
  "query":{
    "constant_score":{
      "filter":{
        "term":{
          "field1.keyword":"..."
        }
      }
    }
  }
}
```

基于全文的查询

```json
POST test/_search
{
  "query":{
    "match":{
      "title":{
        "query": "Matrix reloaded",
        "operator": "AND"
      }
    }
  }
}
```

分页查询

```json
GET test/_search
{
	"query":{
		"match_all":{}
	},
	"from":0,
	"size":2
}
```

带关键字条件查询

```json
GET test/_search
{
	"query":{
		"match":{"name":"兄长"}
	},
}
```

带排序查询

```json
GET test/_search
{
	"query":{
		"match":{"name":"兄长"}
	},
	"sort": [
		{"age":{"order":"desc"}}
	]
}
```

带filter查询

```json
GET test/_search
{
	"query":{
		"bool":{
			"filter":[
				{"term":{"age":30}}, //match:分词
        {"range":{"release_date":{"lte":"2019/11/15"}}}
			]
		}
	}
}
```

带聚合查询

```json
GET test/_search
{
	"query":{
		"match":{"name":"兄长"}
	},
	"sort": [
		{"age":{"order":"desc"}}
	],
	"aggs":{
		"group_by_age":{
			"terms":{
				"field":"age"
			}
		}
	}
}
```

布尔查询

```Json
GET test/_search
{
	"query":{
		"match":{
      "title":{
        "query": "...",
        "operator": "and"
      }
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"match":{
      "title":{
        "query": "...",
        "operator": "or",
        "minimum_should_match": 2
      }
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"bool":{
      "must":[
      	{"match":{"title":""}},
      	{"match":{"overvie":""}}
      ] //shoud, must not。should中为true越多，得分越高
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"bool":{
      "should":[
      	{"match":{"title":""}},
      ],
      "filter":[] //should与filter妙用，用于返回并打分，不满足should，则分数为0
    }
  }
}
```

短语查询

```json
GET test/_search
{
	"query":{
		"match_phrase":{"title":"steven zissou"}
		}
}
```

多字段查询

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title","overview"],
      "operator": "or"
		}	
	}
}
```

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title^10","overview^0.1"],
      "tie_breaker": 0.3 
		}	
	}
}
```

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title","overview"],
			"type":"most_fields"  //sum of, best_fields(default): multiply
		}	
	}
}
```

```json
GET test/_search
{
	"query":{
		"multi_match":{
			"query": "...",
			"fields": ["title","overview"],
			"type":"cross_fields" 
		}	
	}
}
```

QueryString查询

```json
GET test/_search
{
	"query":{
		"query_string":{
			"query": "steve AND jobs",
			"fields": ["title"],
		}	
	}
}
```

自定义打分

```json
GET test/_search
{
	"query":{
		"function_score":{
			"query": {
				"multi_match":{
					"query": "steve job",
					"fields": ["title","overview"],
					"operator": "or",
					"type":"most_fields"  
				}
			},
			"functions": [
				{
					"field_value_factor": {
						"field": "popularity",
						"modifier": "log2p",
						"factor": 5
					}
				}
			],
			"score_mode": "sum", //默认multiply,不同field value得分相加
      "boost_mode": "sum"  //与old value相加
		}	
	}
}
```

```
GET test/_search
{
	"query":{
		"function_score":{
			"query": {
				"bool":{
					"must" 
				}
			},
			"functions": [
				{
					"gauss":{
						"location":{
							"origin":"31.23916171,121.48789949",
							"scale":"100km",
							"offset":"0km",
							"decay":0.5
						}
					}
				},
				{
					"field_value_factor": {
						"field": "remark_score",
					},
					"weight":0.2
				}
			],
			"score_mode": "sum", //默认multiply,不同field value得分相加
      "boost_mode": "sum"  //与old value相加
		}	
	}
}
```

> Mapping

查看mapping

```
GET test/_mappings
```

更改Mapping字段

- 新增加字段
  - Dynamic为true，一旦有新增字段的文档写入，Mapping同时被更新
  - Dynamic为false，Mapping不会更新，新增字段数据无法被索引，但数据会出现在_source中
  - Dynamic设为strict，文档写入失败
- 修改已有字段
  - 不允许修改，必须重建索引

|             | true | false | strict |
| ----------- | ---- | ----- | ------ |
| 文档可索引  | Y    | Y     | N      |
| 字段可索引  | Y    | N     | N      |
| Mapping更新 | Y    | N     | N      |

设置

```json
PUT test
{
  "mappings":{
    "_doc":{
      "dynamic":"false"
    }
  }
}
```

显式Mapping

```json
PUT test
{
  "mappings":{
    "properties":{
      "field1":{
        "type": "..."
      },
      "field2":{
        "type": "...",
        "index": false //不可被搜索
      },
      "mobile":{
        "type": "keyword",
        "null_value": "NULL" //只有Keyword类型支持设定null_value
      },
      "firstName":{
        "type": "text",
        "copy_to": "fullname" //可以搜索fullname,目标字段不出现在_source中，
      }
    }
  }
}
```

```json
GET test/_search
{
	"query":{
		"match":{
			"mobile": "NULL"
		}
	}
}
```

类型

- 数组类型

没有数组type，但可以以[ ]插入多个相同类型数据

- 日期类型

```
{
	"type":"date",
	"format":"8yyyy/MM/dd||yyyy/M/dd||yyyy/MM/d||yyyy/M/d"
}
```

- 对象类型

```
{
	"type":"object",
	"properties":{}
}
```

>  过滤与切分

去除html标签

```json
POST _analyze
{
	"tokenizer":"keyword",
  "char_filter":["html_strip"],
  "text": "<b>hello</b>"
}
```

替换

```json
POST _analyze
{
	"tokenizer":"standard",
  "char_filter":[
  	{
  		"type": "mapping",
  		"mappings": ["- => _"]  //中划线替换为下划线
  	}
  ],
  "text": "123-456, I-test!"
}
```

文件路径切分 

```json
POST _analyze
{
	"tokenizer":"path_hierarchy",
  "text": "/root/user/desktop"
}
```

空格切分

```json
POST _analyze
{
	"tokenizer": "whitespace",
	"filter": ["stop","lowercase"],
  "text": ["The rain is a good rain"]
}
```

自定义

```json
PUT test
{
	"setting":{
		"analysis":{
			"analyzer":{
				"my_analyzer":{
					"type": "custom",
					"char_filter": [
						"emoticons"
					],
					"tokenizer": "punctuation",
					"filter": [
						"lowercase",
						"english_stop"
					]
				}
			},
			"tokenizer":{
				"punctuation":{
					"type":"pattern",
					"pattern": "[.,!?]"
				}
			},
			"char_filter":{
				"emoticons":{
					"type":"mapping",
					"mappings":[
						":)=>_happy_",
						":(=>_sad_"
					]
				}
			},
			"filter":{
				"english_stop":{
					"type": "stop",
					"stopwords": "_english_"
				}
			}
		}
	}
}
```

>  相关性

自定义词典

```xml
<entry key="ext_dict">http://yoursite.com/getCustomDict</entry> 
```

http请求返回两个头部，一个Last-Modified，一个ETag，一个发送变化，词库就会被更新。

同义词

config/analysis-ik放入synonyms.txt

```json
PUT /shop?include_type_name=false
{
   "settings" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 1,
    "analysis": {
      "filter": {
        "my_synonym_filter":{
          "type":"synonym",
          "synonyms_path":"analysis-ik/synonyms.txt"
        }
      },
      "analyzer": {
        "ik_syno":{
          "type":"custom",
          "tokenizer":"ik_smart",
          "filter":["my_synonym_filter"]
        },
        "ik_syno_max":{
          "type":"custom",
          "tokenizer":"ik_max_word",
          "filter":["my_synonym_filter"]
        }
    }  }
   },
   "mappings": {
     "properties": {
       "id":{"type":"integer"},
       "name":{
        "type":"text",
        "analyzer": "ik_syno_max",
        "search_analyzer": "ik_syno" 
       },
       "tags":{"type":"text","analyzer": "whitespace","fielddata":true},
       "location":{"type":"geo_point"},
       "remark_score":{"type":"double"},
       "price_per_man":{"type":"integer"},
       "category_id":{"type":"integer"},
       "category_name":{"type":"keyword"},
       "seller_id":{"type":"integer"},
       "seller_remark_score":{"type":"double"},
       "seller_disabled_flag":{"type":"integer"}
     }
   }
}
```

> 与SpringData整合

- Mybatis

1. 基本用法

打印sql语句

```yaml
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

2. 集成通用Mapper

依赖

```xml
<dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.0.2</version>
</dependency>
```

Mapper

```Java
@Repository
public interface UserMapper extends Mapper<User> {}
```

注解(使用javax.persistence下的注解)

@Table

@Column

@Id 注：必须显式指定Id，否则会选取所有字段当作联合主键

@GeneratedValue 注：让Mapper在执行insert操作后将自动生成的主键写回实体类对象，指定Strategy=GeneratedType.IDENTITY

@Transient 不与实体类字段

方法

selectOne(T t)  使用非空的值生成where子句，实体类结果不能有多个，使用=进行比较

selectByPrimaryKey(Object o)  需要指定主键

selectAll()

selectCount(T t)

***Selective(T t)  非主键字段如果为null，则不加到SQL语句中

QBC查询

```Java
@Test
public void testSelectByExample(){
        Example example = new Example(User.class);
        example.orderBy("credit").desc().orderBy("id").asc();
        example.setDistinct(true);
        example.selectProperties("id", "nickname", "credit");
        Example.Criteria criteria1 = example.createCriteria();
        Example.Criteria criteria2 = example.createCriteria();
        criteria1.andGreaterThan("credit", 60).andLessThan("credit", 80);
        criteria2.andLike("nickname", "onion");
        example.or(criteria2);
        mapper.selectByExample(example);
}
```

3. 使用PageHelper

依赖

```xml
<dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>4.1.4</version>
</dependency>
```

配置

```java
@Bean
public PageHelper pageHelper(){
        PageHelper pageHelper = new PageHelper();
        Properties properties = new Properties();
        properties.setProperty("offsetAsPageNum","true");
        properties.setProperty("rowBoundsWithCount","true");
        properties.setProperty("reasonable","true");
        properties.setProperty("dialect","mysql");    //配置mysql数据库的方言
        pageHelper.setProperties(properties);
        return pageHelper;
}
```

使用

```java
PageHelper.startPage(page, size);
List<Gym> gyms = gymMapper.selectAll();
PageInfo<Gym> pageInfo = new PageInfo<>(gyms);

```

返回信息

```json
{
        "pageNum": 0,
        "pageSize": 5,
        "size": 0,
        "orderBy": null,
        "startRow": 0,
        "endRow": 0,
        "total": 0,
        "pages": 0,
        "list": [],
        "firstPage": 0,
        "prePage": 0,
        "nextPage": 0,
        "lastPage": 0,
        "isFirstPage": false,
        "isLastPage": true,
        "hasPreviousPage": false,
        "hasNextPage": false,
        "navigatePages": 8,
        "navigatepageNums": []
}
```

- Neo4j

> 类型

| Neo4j    | Java             |
| -------- | ---------------- |
| Null     | null             |
| Boolean  |                  |
| Integer  |                  |
| Float    | java.lang.Double |
| String   |                  |
| List     |                  |
| Map      |                  |
| Node     |                  |
| Relation |                  |
| Path     |                  |

> Cypher语言

创建

```cypher
create (n:Person {id:'226',name:'onion',age:20}),(m:Person {id:'227',age:18}),(n)-[:friend]->(m)
match (g:group) where g.groupId = 1 create (u:user {email:"969023025@qq.com",username:"eggplant"})merge(u)-[:join]-(g)
```

查询

```cypher
match (n:Person) return n limit 25

match (n:Person) where n.id='226' return n

match (n:Person {id:'dad'})-[:father]-(f) return n,f //双向，返回儿子和爷爷
match (n:Person {id:'dad'})-[:father]->(f) return n,f //单向，返回儿子

match (n:Person {id:'226'}),(m:Person {id:'227'})
merge (n)-[:boss]->(m) return n,m

match p = (n) --> (m) return p //不限关系
match p = ()-[r:father]->() return type(r),startnode(r),endnode(r)
```

更新

```cypher
match (n:Person {id:'226'}) set n.name = 'mushroom' return n
match (n:Person {id:'227'}) set n.name = 'ys' return n
match(n:old) remove n:old set n:new //修改标签名
match(n)-[r:old]->(m) create(n)-[r2:new]->(m) set r2=r with r delete r//修改关系名
```

删除

```cypher
match (n:Person {id:'227'}) remove n.age return n //remove删除属性和标签label
match (n:Test) remove n:Test

match (t:Teacher)-[r:teach]->(s:Student) delete t,r,s //删节点，若节点有关系，必须先删关系
match (n:Person),(m:Person) where n.id = '226' and m.id = '227'
merge (n)-[r:friend]->(m) delete r

match p=()-[:friend]-() delete p
```

排序、并集、`in`

```cypher
match (n:Person) return n order by n.age desc skip 2 limit 25

match (n:Person) where n.age > 20 return n.id,n.age union all match (n:Person) where n.age < 18 and n.name is not null return n.id, n.age //union会去重

match (n:Person) where n.id in ['226','227'] return n 
```

聚合函数

```cypher
match (n:Person) return count(*)
match (n:Person) return avg(n.age)
match (n:Person) return substring(n.name,0,1) //开始位置，个数
match p = shortestPath((n:Person {id:'mom'})-[*..3]-(b:Person {id:'grandmom'})) return p //3层关系内的最短路径
match p = allshortestPath((n:Person {id:'mom'})-[*..3]-(b:Person {id:'grandmom'})) return p                                                                 
```

其他

```cypher
match (u:user)-[v:view]->(n:note) where u.email = "969023019@qq.com" and n.noteId = "1000002" set v.viewTimes = v.viewTimes + 1
```

初始化与更新实现计数器功能

```cypher
match (u:user),(n:note) where u.email = "969023012@qq.com" and n.noteId = "1000002"   merge (u)-[v:view]-(n) on create set v.viewTimes = 0 on match set v.viewTimes = v.viewTimes + 1
```

案例

查询公交线路全部站点

```cypher
match p=(fro:stop)-[l:line_27*]->(to:stop) where 27 in fro.terminals and 27 in to.terminals return p
```

重命名关系

```cypher
MATCH (n:user)-[f:follow]->(m:user)
CREATE (n)-[f2:fuck]->(m)
SET f2 = f
WITH f
DELETE f
```



> 与SpringData整合

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
```

```yml
spring:
  data:
    neo4j:
      uri: bolt://localhost:7687
      username: neo4j
      password: ***
```



## 第五章： 安全

>  Spring Security

表单认证

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin().and().authorizeRequests().anyRequest().authenticated();
    }
}
```

自定义用户认证

实现`UserDetailsService`接口

```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {
    @Autowired
    private UserDao userDao;
    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        User user = userDao.findByAccount(email);
        return new org.springframework.security.core.userdetails.User(email, user.getPassword(),
                AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_ADMIN"));
    } //加密后的密码
}
```

UserDetails 默认实现类org.springframework.security.core.userdetails.User

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

加密

```java
@Bean
public PasswordEncoder encoder() {
   return new BCryptPasswordEncoder();
}
```

实战

```java
//未登录的处理
public class AjaxAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {

        AjaxResponseBody responseBody = new AjaxResponseBody();

        responseBody.setStatus("000");

        responseBody.setMsg("无访问权限，请先登录");

        httpServletResponse.getWriter().write(JSON.toJSONString(responseBody));
    }
}
//登录失败
public class AjaxAuthenticationFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {

        AjaxResponseBody responseBody = new AjaxResponseBody();

        responseBody.setStatus("400");
       
        responseBody.setMsg("登录失败");

        httpServletResponse.getWriter().write(JSON.toJSONString(responseBody));
    }
}
//登录成功：AuthenticationSuccessHandler
//无权访问：AccessDeniedHandler
//注销 LogoutSuccessHandler 
//自定义对象
public class SelfUserDetails implements UserDetails, Serializable {
    private String username;
    private String password;
    private Set<? extends GrantedAuthority> authorities;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.authorities;
    }

    public void setAuthorities(Set<? extends GrantedAuthority> authorities) {
        this.authorities = authorities;
    }

    @Override
    public String getPassword() { 
        return this.password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
//用户权限认证
public class SelfUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SelfUserDetails userInfo = new SelfUserDetails();
        userInfo.setUsername(username);
        Md5PasswordEncoder md5PasswordEncoder = new Md5PasswordEncoder();
        String encodePassword = md5PasswordEncoder.encodePassword("123", username); 
        userInfo.setPassword(encodePassword);
        Set authoritiesSet = new HashSet();
        GrantedAuthority authority = new SimpleGrantedAuthority("ROLE_ADMIN"); 
        authoritiesSet.add(authority);
        userInfo.setAuthorities(authoritiesSet);
        return userInfo;
    }
}
//前端交互
public class SelfAuthenticationProvider implements AuthenticationProvider {
    @Autowired
    SelfUserDetailsService userDetailsService;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String userName = (String) authentication.getPrincipal();
        String password = (String) authentication.getCredentials(); 

        Md5PasswordEncoder md5PasswordEncoder = new Md5PasswordEncoder();
        String encodePwd = md5PasswordEncoder.encodePassword(password, userName);

        UserDetails userInfo = userDetailsService.loadUserByUsername(userName);

        if (!userInfo.getPassword().equals(encodePwd)) {
            throw new BadCredentialsException("用户名密码不正确，请重新登陆！");
        }

        return new UsernamePasswordAuthenticationToken(userName, password, userInfo.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return true;
    }
}
//全局配置
public class SpringSecurityConf extends WebSecurityConfigurerAdapter {

    @Autowired
    AjaxAuthenticationEntryPoint authenticationEntryPoint;  //  未登陆时返回 JSON 格式的数据给前端（否则为 html）

    @Autowired
    AjaxAuthenticationSuccessHandler authenticationSuccessHandler;  // 登录成功返回的 JSON 格式数据给前端（否则为 html）

    @Autowired
    AjaxAuthenticationFailureHandler authenticationFailureHandler;  //  登录失败返回的 JSON 格式数据给前端（否则为 html）

    @Autowired
    AjaxLogoutSuccessHandler  logoutSuccessHandler;  // 注销成功返回的 JSON 格式数据给前端（否则为 登录时的 html）

    @Autowired
    AjaxAccessDeniedHandler accessDeniedHandler;    // 无权访问返回的 JSON 格式数据给前端（否则为 403 html 页面）

    @Autowired
    SelfAuthenticationProvider provider; // 自定义安全认证

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 加入自定义的安全认证
        auth.authenticationProvider(provider);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http.csrf().disable()

                .httpBasic().authenticationEntryPoint(authenticationEntryPoint)

                .and()
                .authorizeRequests()

                .anyRequest()
                .authenticated()// 其他 url 需要身份认证

                .and()
                .formLogin()  //开启登录
                .loginProcessingUrl("/login")//指定前后端分离的时候调用后台登录接口的名称
                .successHandler(authenticationSuccessHandler) // 登录成功
                .failureHandler(authenticationFailureHandler) // 登录失败
                .permitAll()

                .and()
                .logout()
          			.logoutUrl("/logout")
                .logoutSuccessHandler(logoutSuccessHandler)
                .permitAll();

        http.exceptionHandling().accessDeniedHandler(accessDeniedHandler); // 无权访问 JSON 格式的数据

    }
}

//自定义权限，WebSecurityConfig上面需要加上注解@EnableGlobalMethodSecurity(prePostEnabled = true)
@Component
public class CustomPermissionEvaluator implements PermissionEvaluator {
    /**
     * 自定义验证方法
     * @param authentication        登录的时候存储的用户信息
     * @param targetDomainObject    @PreAuthorize("hasPermission('/hello/**','r')") 中hasPermission的第一个参数
     * @param permission            @PreAuthorize("hasPermission('/hello/**','r')") 中hasPermission的第二个参数
     * @return
     */
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        // 获得loadUserByUsername()方法的结果
        SysUser user = (SysUser)authentication.getPrincipal();
        // 获得loadUserByUsername()中注入的权限
        Collection<? extends GrantedAuthority> authorities = user.getAuthorities();
        // 遍历用户权限进行判定
        for(GrantedAuthority authority : authorities) {
            UrlGrantedAuthority urlGrantedAuthority = (UrlGrantedAuthority) authority;
            String permissionUrl = urlGrantedAuthority.getPermissionUrl();
            // 如果访问的Url和权限用户符合的话，返回true
            if(targetDomainObject.equals(permissionUrl)) {
                return true;
            }
        }
        return false;
    }

    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        return false;
    }
}
```





> Jwt

1. 依赖

```xml
<dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.0</version>
</dependency>
```

2. 工具类

```Java
public class JwtUtil {
    private static final String key = "notehub";
    private static final long ttl = 60 * 60 * 24 * 1000;
    public static String createJwt(User user){
        long now = System.currentTimeMillis();
        JwtBuilder builder = Jwts.builder()
                .setId(user.getId())
                .setIssuedAt(new Date(now))
                .claim("role", user.isAdmin()) //自定义字段
                .claim("nickname",user.getNickname())
                .setExpiration(new Date(now + ttl))
                .signWith(SignatureAlgorithm.HS256, key);
        return builder.compact();
    }
    public static Claims parseJwt(String jwtStr){
        try{
            return Jwts.parser().setSigningKey(key).parseClaimsJws(jwtStr).getBody();
        }catch (Exception e){
            throw new MyException(ResultEnum.INVALID_TOKEN);
        }
    }
}
```

3. 使用AOP验证token

```

```

4. 使用拦截器验证token



## 第六章： 测试

- Junit的用法

```Java
//ParameterizedTest
//@valueSource
public class Exam {
    @ParameterizedTest
    @ValueSource(strings = {"hello world", "enjoy\ncoding"})
    void lowerCase(String candidate){
        assertTrue(StringUtils.containsWhitespace(candidate));
    }
}
//@MethodSource
public class Exam {
    @ParameterizedTest
    @MethodSource("range") //可以指定多个，若不指定，选择同名重载方法
    void exam(int argument){
        assertNotEquals(9, argument);
    }
    static IntStream range(){ //工厂方法，必须static，除非使用@TestInstance(TestInstance.Lifecycle.PER_CLASS)，不能有参数。
        return IntStream.range(0,20).skip(10); //返回值类型：流、数组、集合
    }
}
public class Exam {
    @ParameterizedTest
    @MethodSource("provider")
    void exam(String str, int num, List<String> list){ //多参数，封装到Arguments类
        assertEquals(5,str.length());
        assertTrue(num >= 1 && num <= 2);
        assertEquals(2,list.size());
    }
    static Stream<Arguments> provider(){
        return Stream.of(Arguments.arguments("apple",1, Arrays.asList("a","b")),
                Arguments.arguments("lemon",2, Arrays.asList("x","y")));
    }
}
public class Exam {
    @ParameterizedTest
    @MethodSource("provider")
    void exam(int grade, double finalGrade){
        assertEquals(2.0, grade/finalGrade, 0.0001);
    }
    static Stream<Arguments> provider(){
        return provideInteger().mapToObj(grade->Arguments.of(grade, 0.5*grade));
    }
    static IntStream provideInteger(){
        return IntStream.of(90, 60, 85);
    }
}
//CsvSource
public class Exam {
    @ParameterizedTest
    @CsvSource(delimiter = ',',value = {"apple, 1", "banana, 2"}) //默认逗号分隔
  	//@CsvSource(resources = "xxx.csv", numLinesToSkip = 1)
    void exam(String fruit, int rank){
        assertNotNull(fruit);
        assertNotEquals(0, rank);
    }
}
//其他Source
/*
@NullSource //not applied to primitive type
@EmptySource //String,List,Set,Map,arrays
@NullAndEmptySource //
@EnumSource //value(xxx.class) name and mode(INCLUDE,EXCLUDE,MATCH_ALL,MATCH_ANY) are optional
*/

//Argument Convention

/* 隐式转换
1.提供一个非私有、静态的方法，接收一个参数，返回目标对象的实例
2.提供一个非私有的构造方法，并接收一个参数
*/
/* 显式转换
自定义XXXConverter，继承SimpleArgumentConverter,重写convert方法。
在测试方法的参数加上@ConvertWith(XXXConverter.class)
*/

//自定义DisplayNames
@ParameterizedTest(name="The {index} test: fruit is {0}, rank is {1}")//{index}从1开始；{argument}为完整的，以逗号分隔的参数列表；{0},{1}为每个独立的参数

//TestFactory
@TestFactory
Collection<DynamicTest> dynamicTests(){
   	return Arrays.asList(DynamicTest.dynamicTest("1st",()->assertTrue(true)),
                				 DynamicTest.dynamicTest("2st",()->assertFalse(false)));
}
//使用Stream
public class Exam {
    @TestFactory
    Stream<DynamicTest> testStream(){
        return provideInteger().mapToObj(grade->DynamicTest.dynamicTest("grade: " + grade,()->{assertTrue(grade >= 0 && grade <= 100);}));
    }
    static IntStream provideInteger(){
        return IntStream.of(90, 60, 85);
    }
}
```

- Mockito的用法

spring-boot-starter-test默认集成了Mockito

```java
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;
//spy与mock的区别
class Demo{
    public String foo(){
        return "I like mock";
    }
}
@Test
public void test() {
    Demo demo = spy(Demo.class);
    Demo demo2 = mock(Demo.class); 
    System.out.println(demo.foo()); //I like mock
    System.out.println(demo2.foo()); //null
}
//stub
@Test
public void main() {
    LinkedList mocked = mock(LinkedList.class);
    when(mocked.get(0)).thenReturn("first");
    when(mocked.get(1)).thenThrow(new RuntimeException("Sb"));
    System.out.println(mocked.get(0)); //first
    System.out.println(mocked.get(2)); //null
    System.out.println(mocked.get(1)); //throw exception
  	doThrow(new RuntimeException()).when(mocked).clear();
    mocked.clear();
}
//自定义thenAnswer
Answer<BigDecimal> answer = new Answer<BigDecimal>() {
		@Override
		public BigDecimal answer(InvocationOnMock invocationOnMock) throws Throwable {
          //Object[]args = invocationOnMock.getArguments();
      		//Object mock = invocationOnMock.getMock();
    			return null;
    }
}
//匹配器:若被打桩方法一个参数用了匹配器，则方法所有参数都需要用匹配器
/*
anyInt(),anyLisy(),anyCollection()
isA(T),any(T),eq()
*/
//自定义匹配器
public class Exam {
    @Test
    public void exam() {
        Demo demo = mock(Demo.class);
        when(demo.test(argThat(new MyMatcher()))).thenReturn("bingo");
      	//简写为：when(demo.test(argThat(s->s.equals("test")))).thenReturn("bingo");
        System.out.println(demo.test("test")); //bingo
    }
}
class MyMatcher implements ArgumentMatcher<String>{
    @Override
    public boolean matches(String s) {
        return "test".equals(s);
    }
}
//verify验证方法调用是否符合预期
@Test
public void exam() {
     Demo demo = mock(Demo.class);
     demo.foo("test");
     demo.foo("test again");
     verify(demo).foo(anyString()); //times缺省值为1
     verify(demo, times(1)).foo(anyString()); //fail
  	 //其他mode never(),atLeaseOne(),atLeast(int),only()只调用指定方法
}
//verifyNoMoreInteractions
@Test
public void exam() {
     Demo demo = mock(Demo.class);
     demo.foo("test");
     demo.bar("test");
     verify(demo,times(1)).foo(anyString()); //不能与verifyNoMoreInteractions交换
     verify(demo,times(1)).bar(anyString()); //如果注释掉则不通过
     verifyNoMoreInteractions(demo); //mock对象在测试方法中所有调用都被验证
}
//order
@Test
public void exam() {
     LinkedList list = mock(LinkedList.class);
     list.get(10);
     list.add(10);
     InOrder order = inOrder(list);
     order.verify(list).add(anyInt()); //顺序不对，fail
     order.verify(list).get(anyInt());
}
//连续调用
@Test
public void exam() {
     LinkedList list = mock(LinkedList.class);
     when(list.get(0)).thenReturn(1, -1);//第一次返回1，第二次及以后返回-1
  	 //等价写法：when(list.get(0)).thenReturn(1).thenReturn(-1);
  	 //区别：以下写法为独立打桩，打桩语句使用的方法，如果参数完全一致，后面语句会覆盖前面语句。
  	 /*
  	 when(list.get(0)).thenReturn(1);
  	 when(list.get(0)).thenReturn(-1);
  	 */
     assertEquals(1, list.get(0)); 
     assertEquals(-1, list.get(0));
     assertEquals(-1, list.get(0));
}
```

- 使用mockmvc测试
- 使用mockito进行打桩

```Java
package com.ecnu.timeline.testReconstruct.controller;

import com.alibaba.fastjson.JSON;
import com.ecnu.timeline.controller.NewsController;
import com.ecnu.timeline.domain.News;
import com.ecnu.timeline.service.FileStorageService;
import com.ecnu.timeline.service.NewsService;
import com.ecnu.timeline.vo.Result;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.multipart;

@WebMvcTest(value = NewsController.class)
public class NewsControllerTest {
    @Autowired
    private MockMvc mockMvc;
    @MockBean
    private NewsService newsService;
    @MockBean
    private FileStorageService storageService;
    @Test
    public void shouldGetLatestNews() throws Exception {
        MvcResult mvcResult = mockMvc.perform(get("/latestNews").param("time", "2019-11-11 11:11:11")).andReturn();
        Result result = JSON.parseObject(mvcResult.getResponse().getContentAsByteArray(), Result.class);
        assertEquals(0, (int)result.getCode());
        assertTrue(result.getData() instanceof List);
        assertEquals("查找成功", result.getMessage());
        verify(newsService, times(1)).findLatestNews("2019-11-11 11:11:11");
    }

    @Test
    public void shouldAddNews() throws Exception{
        MvcResult mvcResult = mockMvc.perform(multipart("/add")
                .file("photo", "123".getBytes())
                .param("publisher", "onion")
                .param("content", "内容")
                .param("title", "标题")).andReturn();
        Result result = JSON.parseObject(mvcResult.getResponse().getContentAsByteArray(), Result.class);
        assertEquals(0, (int)result.getCode());
        assertTrue(result.getData() instanceof News);
        assertEquals("发布成功", result.getMessage());
    }

    @Test
    void uploadFile() throws Exception {
        String testFileName = "testFile";
        String downloadUri = "downloadUri";
        when(storageService.storeFile(any())).thenReturn(testFileName);
        when(newsService.addOneFile(anyString(), anyString(), anyInt())).thenReturn(new News().setFileDownloadUri(downloadUri));
        MvcResult mvcResult = mockMvc.perform(
                multipart("/addFile")
                        .file("file", "123".getBytes())
                        .param("newsId", "1")
        ).andReturn();
        Result result = JSON.parseObject(mvcResult.getResponse().getContentAsByteArray(),
                Result.class);
        Integer code = result.getCode();
        String fileDownloadUri = ((News) result.getData()).getFileDownloadUri();
        assertEquals(0, (int) code);
        assertEquals(downloadUri, fileDownloadUri);
    }

}

```

## 第七章： 其他功能
- 邮件发送

依赖：

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

配置文件：

```yaml
mail:
    protocol: smtp
    host: smtp.qq.com
    port: 465
    username: 1452618379@qq.com
    password: tpdvwwqgafgwicae
    default-encoding: utf-8
    properties:
      mail:
        smtp:
          ssl:
            enable: true
    from: 1452618379@qq.com
```

Service类

```java
@Service
@Slf4j
public class MailServiceImpl implements MailService {
    @Autowired
    private JavaMailSender mailSender;
    @Value("${spring.mail.from}")
    private String from;
    @Override
    public void sendMail(String to, String subject, String content) {
        MimeMessage message = mailSender.createMimeMessage();
        MimeMessageHelper helper;
        try {
            helper = new MimeMessageHelper(message,true);
            helper.setFrom(from);
            helper.setSubject(subject);
            helper.setTo(to);
            helper.setText(content,true);
            mailSender.send(message);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }
  @Override
    public void sendHtmlMail(String to, String subject, String text, Map<String, String> attachmentMap) throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        //是否发送的邮件是富文本（附件，图片，html等）
        MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage, true);
        messageHelper.setFrom(from);
        messageHelper.setTo(to);
        messageHelper.setSubject(subject);
        messageHelper.setText(text, true);//默认为false，显示原始html代码，无效果
        if(attachmentMap != null){
            attachmentMap.entrySet().stream().forEach(entrySet -> {
                try {
                    File file = new File(entrySet.getValue());
                    if(file.exists()){
                        messageHelper.addAttachment(entrySet.getKey(), new FileSystemResource(file));
                    }
                } catch (MessagingException e) {
                    e.printStackTrace();
                }
            });
        }
        log.info("发送邮件给{}", to);
        mailSender.send(mimeMessage);
    }
}

```

测试

```java
@Test
    public void testSendFileEmail(){
        Map<String, String> attachmentMap = new HashMap<>();
        attachmentMap.put("\附件名", "\本地文件绝对路径");
        try {
            mailService.sendHtmlMail("\email", "测试Springboot发送带附件的邮件", "欢迎进入<a href=\"http://www.baidu.com\">百度首页</a>", attachmentMap);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }
```

- 定时任务

> 基于@Scheduled

在启动类上添加@EnableScheduling

在方法上添加@Scheduled

fixedRate = 1000 #每隔1秒执行一次，第二次任务开始时第一次任务可能还没结束

fixedDelay #本次任务结束到下次任务开始之间的间隔

initialDelay #首次任务启动的延迟时间

cron表达式：秒、分、时、日、月、周、年(年可以不配置)

| 通配符 | 描述               |
| ------ | ------------------ |
| *      | 任意               |
| ?      | 处理日和星期的冲突 |
| -      | 指定区间           |
| /      | 指定间隔           |
| L      | 最后的             |
| #      | 第几个             |
| ,      | 列举多个           |

举例

| 类型                     | 描述                                |
| ------------------------ | ----------------------------------- |
| 0 0 0 * * ?              | 每天0点整                           |
| 0 15 23 ? * *            | 每天23:15                           |
| 0 * 23 * * ?             | 每天从23点到23点59分每分钟触发一次  |
| 0 0/3 23 * * ?           | 每天从23点到23点59分每3分钟触发一次 |
| 0 0-5 21 * * ?           | 每天21点到21点5分每分钟一次         |
| 0 10,44 14 ? 3 WED       | 三月每周三14:10-14:44触发           |
| 0 0 23 ? * MON-FRI       | 每周一到周五23点出发                |
| 0 30 23 ? * 6L 2017-2020 | 2017-2020每月最后一个周五23:30触发  |
| 0 15 22 ? * 6#3          | 每月第三周周五22:15触发             |

> Quartz

添加依赖

启动类添加@EnableScheduling

定义Job

1. 直接定义bean

```Java
@Component
public class MyJob1(){
  	public void sayHello(){ //存在的缺陷：无法传参
      	System.out.println("hello");
    }
}
```

2. 继承

```java
public class MyJob2 extends QuartzJobBean {
    HelloService helloService;
    public HelloService getHelloService() {
        return helloService;
    }
    public void setHelloService(HelloService helloService) {
        this.helloService = helloService;
    }
    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        helloService.sayHello();
    }
}
public class HelloService {
    public void sayHello() {
        System.out.println("hello service >>>"+new Date());
    }
}
```

3. 配置

```java
@Configuration
public class QuartzConfig {
  //JobDetail 的配置有两种方式：MethodInvokingJobDetailFactoryBean 和 JobDetailFactoryBean 。
    @Bean //使用 MethodInvokingJobDetailFactoryBean 可以配置目标 Bean 的名字和目标方法的名字，这种方式不支持传参。
    MethodInvokingJobDetailFactoryBean methodInvokingJobDetailFactoryBean() {
        MethodInvokingJobDetailFactoryBean bean = new MethodInvokingJobDetailFactoryBean();
        bean.setTargetBeanName("myJob1");
        bean.setTargetMethod("sayHello");
        return bean;
    }
    @Bean //使用 JobDetailFactoryBean 可以配置 JobDetail ，任务类继承自 QuartzJobBean ，这种方式支持传参，将参数封装在 JobDataMap 中进行传递。
    JobDetailFactoryBean jobDetailFactoryBean() {
        JobDetailFactoryBean bean = new JobDetailFactoryBean();
        bean.setJobClass(MyJob2.class);
        JobDataMap map = new JobDataMap();
        map.put("helloService", helloService());
        bean.setJobDataMap(map);
        return bean;
    }
  //Trigger 是指触发器，Quartz 中定义了多个触发器，这里向大家展示其中两种的用法，SimpleTrigger 和 CronTrigger 。
    @Bean
    SimpleTriggerFactoryBean simpleTriggerFactoryBean() {
        SimpleTriggerFactoryBean bean = new SimpleTriggerFactoryBean();
        bean.setStartTime(new Date());
        bean.setRepeatCount(5);
        bean.setJobDetail(methodInvokingJobDetailFactoryBean().getObject());
        bean.setRepeatInterval(3000);
        return bean;
    }
    @Bean
    CronTriggerFactoryBean cronTrigger() {
        CronTriggerFactoryBean bean = new CronTriggerFactoryBean();
        bean.setCronExpression("0/10 * * * * ?");
        bean.setJobDetail(jobDetailFactoryBean().getObject());
        return bean;
    }
    @Bean
    SchedulerFactoryBean schedulerFactoryBean() {
        SchedulerFactoryBean bean = new SchedulerFactoryBean();
        bean.setTriggers(cronTrigger().getObject(), simpleTriggerFactoryBean().getObject());
        return bean;
    }
    @Bean
    HelloService helloService() {
        return new HelloService();
    }
}
```

- 数据校验

依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```



## 第八章：经典问题解决

## 第九章：部署

> docker

### 命令

#### 帮助命令

```shell
docker version
docker info
docker --help
```

#### 镜像命令

```shell
docker images
-a #列出本地所有的镜像（含中间映像层）
-q #只显示镜像ID
--digests #显示镜像的摘要信息
-no-trunc #显示完整的镜像信息

docker search 镜像名
docker pull 镜像名[:TAG]
docker rmi 镜像名
#单个：docker rmi  -f 镜像ID
#多个：docker rmi -f 镜像名1:TAG 镜像名2:TAG
#全部：docker rmi -f $(docker images -qa)

docker commit -m=“提交的描述信息” -a=“作者” 容器ID 要创建的目标镜像名:[标签名]

#push到公有仓库
docker login
docker push onion12138/hello:latest

#push到私有仓库
docker run -d -p 5000:5000 --name registry registry #120.77.176.55上运行
docker build -t 120.77.176.55:5000/hello-world:latest .
docker push 120.77.176.55:5000/hello-world
//需要在客户端修改
	{
    "registry-mirrors": ["https://registry.docker-cn.com","http://hub-mirror.c.163.com"],
    "insecure-registries": ["120.77.176.55:5000"]
}
//使用API查看
http://120.77.176.55:5000/v2/_catalog
//测试是否成功
docker rmi 120.77.176.55:5000/hello-world:latest
docker pull 120.77.176.55:5000/hello-world
```

#### 容器命令

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
--name="容器新名字": 为容器指定一个名称； 
-d: 后台运行容器，并返回容器ID，也即启动守护式容器； 
-i：以交互模式运行容器，通常与 -t 同时使用； 
-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用； 
-P: 随机端口映射； 
-p: 指定端口映射，有以下四种格式 
      ip:hostPort:containerPort 
      ip::containerPort 
      hostPort:containerPort 
      containerPort 

docker ps
#退出
exit 容器停止退出
ctrl+P+Q 容器不停止退出
#操作
docker start/restart/stop/kill 容器ID或者容器名
#删除
docker rm 容器ID 
#一次性删除多个
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
#查看日志
docker logs -f -t --tail 容器ID
 -t 是加入时间戳
 -f 跟随最新的日志打印
 --tail 数字 显示最后多少条
#查看容器内运行的进程
docker top 容器ID
#查看容器内部细节
docker inspect 容器ID
#进入正在运行的容器并以命令行交互
docker exec -it 容器ID bashShell
#重新进入
docker attach 容器ID
#区别
attach 直接进入容器启动命令的终端，不会启动新的进程
exec 是在容器中打开新的终端，并且可以启动新的进程
#从容器内拷贝文件到主机上
docker cp  容器ID:容器内路径 目的主机路径

```

### 数据卷

```shell
#通过命令添加
docker run -it -v /宿主机绝对路径目录:/容器内目录      镜像名
docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名 #只读
#通过dockerfile添加，指定的为容器目录，使用inspect查看主机目录
VOLUME["/dataVolumeContainer","/dataVolumeContainer2","/dataVolumeContainer3"] 
#Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied的解决办法：在挂载目录后多加一个--privileged=true参数即可 

#volumes-from
docker run -it --name dc02 --volumes-from dc01 zzyy/centos 
```

### Dockerfile

```dockerfile
#from
from scratch
from centos
#label
label maintainer="onion@qq.com" 
#run 命令,反斜线换行 
#workdir 工作目录
workdir /test
workdir demo
run pwd #/test/demo
#add
add hello /
add test.tar.gz / #添加并解压
#copy
workdir /root
copy hello test/ #大部分情况copy优于add，add还有解压功能，添加远程文件/目录使用curl或wget
#env
env MYSQL_VERSION 5.6 #设置常量
run apt-get install -y mysql-server = "${MYSQL_VERSION}" \
&& rm -rf /var/lib/apt/lists/*
#cmd 容器启动时默认执行的命令；如果docker run指定了其他命令，cmd命令会被忽略；定义多个cmd，只执行最后一个

#entrypoint 让容器以应用程序或服务的形式运行，不会被忽略，最佳实践为写一个shell脚本作为entrypoint
copy docker-entrypoint.sh /usr/local/bin/
entrypoint ["docker-entrypoint.sh"]
expose 27017
cmd ["mongod"]

from centos
env name Docker
entrypoint echo "hello $name" 

from centos
env name Docker
entrypoint ["/bin/bash","-c", "echo hello $name"] 


#创建
docker build -t 新镜像名字:TAG .
```

> Nginx

### 安装

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

1. `Master`进程

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

2. `Worker`进程

设置`worker`进程个数：

`worker_processes auto`  

接收信号：

- `TERM`，`INT`
- `QUIT`
- `USR1`
- `WINCH`

3. `Nginx`命令行

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

### 实战部署前后端分离项目

在宿主机建立目录nginx/www, nginx/conf, nginx/logs

```shell
docker run -di -p 80:80 --name nginx -v ~/nginx/www:/usr/share/nginx/html -v ~/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v ~/nginx/logs:/var/log/nginx nginx 
```

配置文件

```nginx
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  120.77.176.55;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        location /api/ {
                proxy_pass http://120.77.176.55:8080/;
                add_header Access-Control-Allow-Methods *;
                add_header Access-Control-Max-Age 3600;
                add_header Access-Control-Allow-Credentials true;
                add_header Access-Control-Allow-Origin $http_origin;
                add_header Access-Control-Allow-Headers $http_access_control_request_headers;
                if ($request_method = OPTIONS){
                        return 200;
                }
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

将打包后的前端文件放入nginx/www/目录下

