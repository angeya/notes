## Linux下安装

1. 下载软件包mongodb-linux-x86_64-rhel70-6.0.3.tgz，并解压到 /usr/local/mongodb文件夹。

2. 创建日志目录 /usr/local/mongodb/single/log，数据目录 /usr/local/mongodb/single/data/db。

3. 在 /usr/local/mongodb/ 下创建配置文件 mongodb.conf，并输入以下内容：

   ```yaml
   systemLog:
      destination: file
      path: "/usr/local/mongodb/single/log/mongod.log" # 设置日志文件路径
      logAppend: true
   storage:
      dbPath: "/usr/local/mongodb/single/data/db" # 设置数据存储路径
      journal:
         enabled: true # 启用或禁用持久性日志以确保数据文件保持有效和可恢复。
   processManagement:
      fork: true # 后台运行
   net:
      bindIp: 0.0.0.0
      port: 27017
   setParameter:
      enableLocalhostAuthBypass: false
   security:
      authorization: enabled # 开启登录验证功能
   ```

4. 使用以下命令启动mongodb服务：`mongod -f /usr/local/mongodb/mongod.conf`

## 连接数据库

连接数据库可以使用命令行方式连接，也可以使用官方提供GUI工具的Compass进行连接。

命令可以通过连接字符串 `mongodb://admin:123456=@121.4.16.58/admin` 或者 `mongo admin -u admin -p 123456` 命令进行连接。

## 设置登录验证

1. 使用compass工具进行连接数据库（将 security.authorization: enabled 注掉才能连上）

2. 使用 use admin 命令进入admin数据库（当然，也可以指定别的库，admin库包含所有库的权限）

3. 使用如下命令创建用户

   ```bash
   db.createUser({
     user: 'admin',  // 用户名
     pwd: '123ASDasd=',  // 密码
     roles:[{
       role: 'admin',  // 角色
       db: 'admin'  // 数据库
     }]
   })
   ```

4. 使用 db.auth("user", "password") 命令查看验证是否成功。成功之后，放开mongodb服务 security.authorization: enabled 配置并重启即可。

## MongoDB常用命令

选择和切换数据库

```bash
use test 
```

插入数据

```bash
db.comment.insert({bson数据})
```

查询所有数据

```bash
db.user.find(); 
```

条件查询数据

```bash
db.user.find({条件}) 
```

查询符合条件的第一条记录

```bash
db.user.findOne({条件}) 
```

查询符合条件的前几条记录

```bash
db.user.find({条件}).limit(条数) 
```

查询符合条件的跳过的记录

```bash
db.user.find({条件}).skip(条数) 
```

修改数据

```bash
db.user.update({条件},{修改后的数据}) 或db.user.update({条件},{$set:{要修改部分的字段:数据}) 
```

修改数据并自增某字段值

```bash
db.user.update({条件},{$inc:{自增的字段:步进值}}) 
```

删除数据

```bash
db.user.remove({条件}) 
```

统计查询

```bash
db.user.count({条件}) 
```

模糊查询

```bash
db.user.find({字段名:/正则表达式/}) 
```

条件比较运算

```bash
db.user.find({字段名:{$gt:值}}) 
```

包含查询

```bash
db.user.find({字段名:{$in:[值1，值2]}})或db.user.find({字段名:{$nin:[值1，值2]}}) 
```

根据日期、模块、登记分组分别查询慢sql日志
```bash
// 日期
db.pf_slow_sql_log.aggregate([
  {
    $project: {
      // 处理字段 creastamp
      creastampDay: {
        $dateToString: { format: "%Y-%m-%d", date: "$creastamp" }
      }
    }
  },
  {
    $group: {
      _id: {
        creastampDay: "$creastampDay"
      },
      count: { $sum: 1 }
    }
  },
  {
    $sort: { "_id.creastampDay": 1}
  }
])

-- 模块

db.pf_slow_sql_log.aggregate([
  {
    $group: {
      _id: {
        module: "$module",
      },
      count: { $sum: 1 }
    }
  },
  {
    $sort: {"_id.module": 1}
  }
])

-- 级别
db.pf_slow_sql_log.aggregate([
  {
    // 处理duration字段
    $project: {
      durationGroup: {
        // 根据duration的值，分配500、1000、2000
        $cond: {
          if: { $lt: ["$duration", 1000] }, // 如果duration小于1000
          then: {
            $cond: {
              if: { $lt: ["$duration", 500] }, // 如果duration小于500
              then: '<500',
              else: '500-1000'
            }
          },
          else: {
            $cond: {
              if: { $lt: ["$duration", 2000] }, // 如果duration小于2000
              then: '1000-2000',
              else: '>=2000'
            }
          }
        }
      }
    }
  },
  {
    // 按durationGroup分组
    $group: {
      _id: {
        durationGroup: "$durationGroup"
      },
      count: { $sum: 1 }
    }
  },
  {
    $sort: {"_id.durationGroup": 1 }
  }
])
```



## SpringBoot整合MongoDB

### 环境准备

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
       <version>2.5.14</version>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-mongodb</artifactId>
       <version>2.5.14</version> 
   </dependency>
   ```

2. 编写配置文件application.yml

   ```yaml
   spring:
     data:
       mongodb:
         host: 121.4.16.58
         database: test
         port: 27017
         username: admin
         password: 123456
   ```

   

3. 定义数据库实体类User

   ```java
   // 注解可以省略，如果省略，则默认使用类名小写映射文档对应的集合
   @Document(collection="user")
   public class User {
       
       //主键标识，该属性的值会自动对应mongodb的主键字段"_id"，如果该属性名就叫“id”，则该注解可以省略
       @Id
       private String id;
   
       //该属性对应mongodb的字段的名字，如果一致，则该注解可以省略
       @Field("content")
       private String name;
   
       private Integer age;
   
       // @Indexed(name = "user_city_idx") // 添加索引
       private String city;
       
       // 省略Setter和Getter
   }
   ```
   
   > spring-boot设置中未将**auto-index-creation**设为true, 从**Spring Data MongoDB 3.0**以后，出于防止滥用的考虑，自动创建索引是默认关闭的，需要设置开启。配置项为：spring.data.mongodb.auto-index-creation=true。
   >
   > 可以通过如下命令查看集合的索引(假设集合名称是user：db.user.getIndexes()

### 类JPA的方式

1. 编写Mapper

   ```java
   public interface MongoMapper extends MongoRepository<User, String> {
       /**
        * 自定义方法：根据年龄查询用户分页列表
        * @param age 年龄
        * @param pageable 分页信息
        * @return 分页数据
        */ 
       Page<User> findByAge(Integer age, Pageable pageable);
   }
   ```

2. Service层的调用

   ```java
   @Service
   public class MongoService {
   
       @Autowired
       private MongoMapper mongoMapper;
   
       public Page<User> findUserPageByAge(Integer age, int page , int size){
           return mongoMapper.findByAge(age, PageRequest.of(page-1,size));
       }
   
       public List<User> getAll() {
           return this.mongoMapper.findAll();
       }
   
       public User getById(String id) {
           return this.mongoMapper.findById(id).get();
       }
   
       public void save(User user) {
           this.mongoMapper.insert(user);
       }
   
       public void update(User user) {
           this.mongoMapper.save(user);
       }
   
       public void delete(String id) {
           this.mongoMapper.deleteById(id);
       }
   }
   ```

   

### MongoTemplate 方式

#### 基本用法

直接引入MongoTemplate并使用，使用方式类似MybatisPlus

```java
@Service
public class MongoService {

    @Autowired
    private MongoTemplate mongoTemplate;

    public void increaseAge(String id) {
        // 查询对象，用于确认修改哪一个文档
        Query query = Query.query(Criteria.where("_id").is(id));
        // 更新对象，包含具体更新操作
        Update update = new Update();
        // 更新增加的的字段，可以通过第二个参数设置步长
        update.inc("age");
        this.mongoTemplate.updateFirst(query, update, "user");
    }
}
```

#### 增删改查

1. 新增数据

   ```java
   mongoTemplate.insert(user);
   mongoTemplate.save(user);
   ```

   使用 `insert()` 方法将 `User` 对象插入到 MongoDB 中，如果该对象ID已存在，则不会再次插入。可以使用 `save()` 方法实现相同的功能，但是它也会替换掉已经存在的文档（值为null的字段则被删除），因此更适合执行覆盖性的操作。

2. 删除

   ```java
   Query query = new Query(Criteria.where("id").is(id));
   mongoTemplate.remove(query, User.class);
   ```

   使用 `remove()` 方法根据 ID 删除单个 `User` 对象，可以使用 `removeMulti()` 方法实现删除多个文档。

3. 修改

   ```java
   Query query = new Query(Criteria.where("age").is(age));
   Update update = new Update().set("name", name);
   mongoOps.updateMulti(query, update, User.class);
   ```

4. 查询

   ```java
   Query query = new Query(Criteria.where("id").is(id));
   return mongoTemplate.findOne(query, User.class);
   ```

   



