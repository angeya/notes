## Mybatis

一、if：你们能判断，我也能判断！

```sql
<select id="count" resultType="java.lang.Integer">
	select count(*) from user where <if test="id != null">id = #{id}</if> and username = 'xiaoming'
</select>
原文链接：https://blog.csdn.net/qq_39249094/article/details/107199696
```

如果传入的 id 不为 null， 那么才会 SQL 才会拼接 id = #{id}
如果传入的 id 为 null，那么最终的 SQL 语句就变成了 select count(*) from user where and username = ‘xiaoming’。这语句就会有问题，这时候 where 标签就该隆重登场了
二、where：有了我，SQL 语句拼接条件神马的都是浮云！

```sql
<select id="count" resultType="java.lang.Integer">
	select count(*) from user
	<where>
		<if test="id != null">id = #{id}</if>
		and username = 'xiaoming'
    </where>
 </select>
```

where 元素只会在至少有一个子元素的条件返回 SQL 子句的情况下才去插入 WHERE 子句
若语句的开头为 AND、OR，where 元素也会将它们去除。还可以通过 trim 标签去自定义这种处理规则
三、trim：我的地盘，我做主！
trim 标签一般用于拼接、去除 SQL 的前缀、后缀
trim 标签中的属性
属性	描述
prefix	拼接前缀
suffix	拼接后缀
prefixOverrides	去除前缀
suffixOverrides	去除后缀

```sql
<select id="count" result="java.lang.Integer">
	select count(*) from user
	<trim prefix ="where" prefixOverrides="and | or">
		<if test="id != null">id = #{id}</if>
		<if test="username != null"> and username = #{username}</if>
	</trim>
</select>
```



如果 id 或者 username 有一个不为空，则在语句前加入 where。如果 where 后面紧随 and 或 or 就会自动会去除
如果 id 或者 username 都为空，则不拼接任何东西
四、set： 信我，不出错！

```sql
<update id="UPDATE" parameterType="User">
	update user
    <set>
    	<if test="name != null">name = #{name},</if> 
        <if test="password != null">password = #{password},</if> 
        <if test="age != null">age = #{age},</if> 
   	</set>
</update>
```

三个 if 至少有一个不为空。会在前面加上 set，自动去除尾部多余的逗号
五、foreach: 你有 for，我有 foreach
foreach 标签中的属性
属性	描述
index	下标
item	每个元素名称
open	该语句以什么开始
close	该语句以什么结尾
separator	在每次迭代之间以什么作为分隔符
collection	参数类型
collection：
如果参数类型为 List，则该值为 list

```sql
<select id="count" resultType="java.lang.Integer">
	select count(*) from user where id in
  	<foreach collection="list" item="item" index="index" open="(" separator="," close=")">
        #{item}
  	</foreach>
</select>
```

如果参数类型为数组，则该值为 array

```sql
<select id="count" resultType="java.lang.Integer">
	select * from user where id inarray
  	<foreach collection="array" item="item" index="index" open="(" separator="," close=")">
        #{item}
  	</foreach>
</select>
```

如果参数类型为 Map，则参数类型为 Map 的 Key
六、choose: 我选择了你，你选择了我！

```sql
<select id="count" resultType="Blog">
	select count(*) from user
  	<choose>
    	<when test="id != null">
      		and id = #{id}
    	</when>
    	<when test="username != null">
      		and username = #{username}
    	</when>
    	<otherwise>
      		and age = 18
    	</otherwise>
  	</choose>
</select>
```

当 id 和 username 都不为空的时候， 那么选择二选一（前者优先）
如果都为空，那么就选择 otherwise 中的
如果 id 和 username 只有一个不为空，那么就选择不为空的那个
七、sql：相当于 Java 中的代码提重，需要配合 include 使用

```sql
<sql id="table"> user </sql>
```

八、include：相当于 Java 中的方法调用

```sql
<select id="count" resultType="java.lang.Integer">
	select count(*) from <include refid=“table（sql 标签中的 id 值）” />
</select>
```



九、bind：对数据进行再加工

```sql
<select id="count" resultType="java.lang.Integer">
	select count(*) from user
	<where>
		<if test="name != null">
			<bind name="name" value="'%' + username + '%'"
			name = #{name}
		</if>
</select>
```





## Mybatis Plus

### 简介

[MyBatis-Plus (opens new window)](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis (opens new window)](https://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

#### 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

#### 支持数据库)支持数据库

> 任何能使用 `MyBatis` 进行 CRUD, 并且支持标准 SQL 的数据库，具体支持情况如下，如果不在下列表查看分页部分教程 PR 您的支持。

- MySQL，Oracle，DB2，H2，HSQL，SQLite，PostgreSQL，SQLServer，Phoenix，Gauss ，ClickHouse，Sybase，OceanBase，Firebird，Cubrid，Goldilocks，csiidb，informix，TDengine，redshift
- 达梦数据库，虚谷数据库，人大金仓数据库，南大通用(华库)数据库，南大通用数据库，神通数据库，瀚高数据库，优炫数据库，星瑞格数据库

#### 框架结构

<img src="https://baomidou.com/img/mybatis-plus-framework.jpg" alt="framework" style="zoom:50%;" />

### 基础项目搭建

#### pom依赖

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.9.RELEASE</version>
    </parent>

    <dependencies>
        <!-- springBoot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <!-- mybatis plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.2</version>
        </dependency>

        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.27</version>
        </dependency>

        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```

#### mapperScan注解

```java
@SpringBootApplication
@MapperScan("top.angeya.dao")
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

#### 实体类User

```java
@TableName("user")
@Data
public class User  {

    // 必须要要有空构造器，否则MP框架无法构建查询结果
    public User() {}

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    private String id;

    private String name;

    private Integer age;

    private String city;

    private String uniId;
}
```

#### mapper层

```java
@Repository
public interface UserMapper extends BaseMapper<User> {
}
```

#### service层

```java
@Service
public class UserService extends ServiceImpl<UserMapper, User> {

}
```



#### 单元测试类

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import top.angeya.entity.User;

import java.util.List;

/**
 * @author wanganjie 5790
 * @since 2023/10/13 16:02
 */
@SpringBootTest
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void testList() {
        List<User> userList = userService.list();
        userList.forEach(System.out::println);
    }
}
```

### 语法

#### Service CRUD 接口
说明:

通用 Service CRUD 封装IService (opens new window)接口，进一步封装 CRUD 采用 get 查询单行 remove 删除 list 查询集合 page 分页 前缀命名方式区分 Mapper 层避免混淆，
泛型 T 为任意实体对象
建议如果存在自定义通用 Service 方法的可能，请创建自己的 IBaseService 继承 Mybatis-Plus 提供的基类
对象 Wrapper 为 条件构造器

##### Save
```java
// 插入一条记录（选择字段，策略插入）
boolean save(T entity);
// 插入（批量）
boolean saveBatch(Collection<T> entityList);
// 插入（批量）
boolean saveBatch(Collection<T> entityList, int batchSize);
```

#参数说明
类型	参数名	描述
T	entity	实体对象
Collection<T>	entityList	实体对象集合
int	batchSize	插入批次数量

##### SaveOrUpdate

```java
// TableId 注解存在更新记录，否插入一条记录
boolean saveOrUpdate(T entity);
// 根据updateWrapper尝试更新，否继续执行saveOrUpdate(T)方法
boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList);
// 批量修改插入
boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
```

#参数说明
类型	参数名	描述
T	entity	实体对象
Wrapper<T>	updateWrapper	实体对象封装操作类 UpdateWrapper
Collection<T>	entityList	实体对象集合
int	batchSize	插入批次数量

##### Remove

```java
// 根据 queryWrapper 设置的条件，删除记录
boolean remove(Wrapper<T> queryWrapper);
// 根据 ID 删除
boolean removeById(Serializable id);
// 根据 columnMap 条件，删除记录
boolean removeByMap(Map<String, Object> columnMap);
// 删除（根据ID 批量删除）
boolean removeByIds(Collection<? extends Serializable> idList);
```

#参数说明
类型	参数名	描述
Wrapper<T>	queryWrapper	实体包装类 QueryWrapper
Serializable	id	主键 ID
Map<String, Object>	columnMap	表字段 map 对象
Collection<? extends Serializable>	idList	主键 ID 列表

##### Update

```java
// 根据 UpdateWrapper 条件，更新记录 需要设置sqlset
boolean update(Wrapper<T> updateWrapper);
// 根据 whereWrapper 条件，更新记录
boolean update(T updateEntity, Wrapper<T> whereWrapper);
// 根据 ID 选择修改
boolean updateById(T entity);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList);
// 根据ID 批量更新
boolean updateBatchById(Collection<T> entityList, int batchSize);
```

#参数说明
类型	参数名	描述
Wrapper<T>	updateWrapper	实体对象封装操作类 UpdateWrapper
T	entity	实体对象
Collection<T>	entityList	实体对象集合
int	batchSize	更新批次数量

##### Get

```java
// 根据 ID 查询
T getById(Serializable id);
// 根据 Wrapper，查询一条记录。结果集，如果是多个会抛出异常，随机取一条加上限制条件 wrapper.last("LIMIT 1")
T getOne(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
T getOne(Wrapper<T> queryWrapper, boolean throwEx);
// 根据 Wrapper，查询一条记录
Map<String, Object> getMap(Wrapper<T> queryWrapper);
// 根据 Wrapper，查询一条记录
<V> V getObj(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```

#参数说明
类型	参数名	描述
Serializable	id	主键 ID
Wrapper<T>	queryWrapper	实体对象封装操作类 QueryWrapper
boolean	throwEx	有多个 result 是否抛出异常
T	entity	实体对象
Function<? super Object, V>	mapper	转换函数

##### List

```java
// 查询所有
List<T> list();
// 查询列表
List<T> list(Wrapper<T> queryWrapper);
// 查询（根据ID 批量查询）
Collection<T> listByIds(Collection<? extends Serializable> idList);
// 查询（根据 columnMap 条件）
Collection<T> listByMap(Map<String, Object> columnMap);
// 查询所有列表
List<Map<String, Object>> listMaps();
// 查询列表
List<Map<String, Object>> listMaps(Wrapper<T> queryWrapper);
// 查询全部记录
List<Object> listObjs();
// 查询全部记录
<V> List<V> listObjs(Function<? super Object, V> mapper);
// 根据 Wrapper 条件，查询全部记录
List<Object> listObjs(Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录
<V> List<V> listObjs(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```

#参数说明
类型	参数名	描述
Wrapper<T>	queryWrapper	实体对象封装操作类 QueryWrapper
Collection<? extends Serializable>	idList	主键 ID 列表
Map<String, Object>	columnMap	表字段 map 对象
Function<? super Object, V>	mapper	转换函数

##### Page

```
// 无条件分页查询
IPage<T> page(IPage<T> page);
// 条件分页查询
IPage<T> page(IPage<T> page, Wrapper<T> queryWrapper);
// 无条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page);
// 条件分页查询
IPage<Map<String, Object>> pageMaps(IPage<T> page, Wrapper<T> queryWrapper);
#参数说明
类型	参数名	描述
IPage<T>	page	翻页对象
Wrapper<T>	queryWrapper	实体对象封装操作类 QueryWrapper
```

##### Count

```
// 查询总记录数
int count();
// 根据 Wrapper 条件，查询总记录数
int count(Wrapper<T> queryWrapper);
#参数说明
类型	参数名	描述
Wrapper<T>	queryWrapper	实体对象封装操作类 QueryWrapper
```

##### Chain
###### query
```
// 链式查询 普通
QueryChainWrapper<T> query();
// 链式查询 lambda 式。注意：不支持 Kotlin
LambdaQueryChainWrapper<T> lambdaQuery();

// 示例：
query().eq("column", value).one();
lambdaQuery().eq(Entity::getId, value).list();
```

###### update

```
// 链式更改 普通
UpdateChainWrapper<T> update();
// 链式更改 lambda 式。注意：不支持 Kotlin
LambdaUpdateChainWrapper<T> lambdaUpdate();

// 示例：
update().eq("column", value).remove();
lambdaUpdate().eq(Entity::getId, value).update(entity);
```

#### Mapper CRUD 接口
说明:

通用 CRUD 封装BaseMapper (opens new window)接口，为 Mybatis-Plus 启动时自动解析实体表关系映射转换为 Mybatis 内部对象注入容器
泛型 T 为任意实体对象
参数 Serializable 为任意类型主键 Mybatis-Plus 不推荐使用复合主键约定每一张表都有自己的唯一 id 主键
对象 Wrapper 为 条件构造器
#Insert
// 插入一条记录
int insert(T entity);
#参数说明
类型	参数名	描述
T	entity	实体对象

#Delete
// 根据 entity 条件，删除记录
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
// 删除（根据ID 批量删除）
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 ID 删除
int deleteById(Serializable id);
// 根据 columnMap 条件，删除记录
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
#参数说明
类型	参数名	描述
Wrapper<T>	wrapper	实体对象封装操作类（可以为 null）
Collection<? extends Serializable>	idList	主键 ID 列表(不能为 null 以及 empty)
Serializable	id	主键 ID
Map<String, Object>	columnMap	表字段 map 对象

#Update
// 根据 whereWrapper 条件，更新记录
int update(@Param(Constants.ENTITY) T updateEntity, @Param(Constants.WRAPPER) Wrapper<T> whereWrapper);
// 根据 ID 修改
int updateById(@Param(Constants.ENTITY) T entity);
使用提示:

在调用updateById方法前，需要在T entity（对应的实体类）中的主键属性上加上@TableId注解。

#参数说明
类型	参数名	描述
T	entity	实体对象 (set 条件值,可为 null)
Wrapper<T>	updateWrapper	实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）

#Select
// 根据 ID 查询
T selectById(Serializable id);
// 根据 entity 条件，查询一条记录
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 查询（根据ID 批量查询）
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
// 根据 entity 条件，查询全部记录
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 查询（根据 columnMap 条件）
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
// 根据 Wrapper 条件，查询全部记录
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录。注意： 只返回第一个字段的值
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据 entity 条件，查询全部记录（并翻页）
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录（并翻页）
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询总记录数
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
#参数说明
类型	参数名	描述
Serializable	id	主键 ID
Wrapper<T>	queryWrapper	实体对象封装操作类（可以为 null）
Collection<? extends Serializable>	idList	主键 ID 列表(不能为 null 以及 empty)
Map<String, Object>	columnMap	表字段 map 对象
IPage<T>	page	分页查询条件（可以为 RowBounds.DEFAULT）

#### mapper 层 选装件

说明:

选装件位于 com.baomidou.mybatisplus.extension.injector.methods 包下 需要配合Sql 注入器使用,案例(opens new window)
使用详细见源码注释(opens new window)

##### AlwaysUpdateSomeColumnById(opens new window)

```java
int alwaysUpdateSomeColumnById(T entity);
```

##### insertBatchSomeColumn(opens new window)

```java
int insertBatchSomeColumn(List<T> entityList);
```

##### logicDeleteByIdWithFill(opens new window)

```java
int logicDeleteByIdWithFill(T entity);
```

#### ActiveRecord 模式

说明:

实体类只需继承 Model 类即可进行强大的 CRUD 操作
需要项目中已注入对应实体的BaseMapper
#操作步骤：
继承 Model(opens new window)

```java
class User extends Model<User>{
    // fields...
}
```

调用CRUD方法(演示部分api，仅供参考)

```java
User user = new User();
user.insert();
user.selectAll();
user.updateById();
user.deleteById();
// ...
```

#### SimpleQuery 工具类
说明:

对selectList查询后的结果用Stream流进行了一些封装，使其可以返回一些指定结果，简洁了api的调用
需要项目中已注入对应实体的BaseMapper
使用方式见: 测试用例(opens new window)
对于下方参数peeks，其类型为Consumer...，可一直往后叠加操作例如：List<Long> ids = SimpleQuery.list(Wrappers.lambdaQuery(), Entity::getId, System.out::println, user -> userNames.add(user.getName()));
##### keyMap
```java
// 查询表内记录，封装返回为Map<属性,实体>
Map<A, E> keyMap(LambdaQueryWrapper<E> wrapper, SFunction<E, A> sFunction, Consumer<E>... peeks);
// 查询表内记录，封装返回为Map<属性,实体>，考虑了并行流的情况
Map<A, E> keyMap(LambdaQueryWrapper<E> wrapper, SFunction<E, A> sFunction, boolean isParallel, Consumer<E>... peeks);
```

#参数说明
类型	参数名	描述
E	entity	实体对象
A	attribute	实体属性类型,也是map中key的类型
LambdaQueryWrapper<E>	wrapper	支持lambda的条件构造器
SFunction<E, A>	sFunction	实体中属性的getter,用于封装后map中作为key的条件
boolean	isParallel	为true时底层使用并行流执行
Consumer<E>...	peeks	可叠加的后续操作

##### map
```java
// 查询表内记录，封装返回为Map<属性,属性>
Map<A, P> map(LambdaQueryWrapper<E> wrapper, SFunction<E, A> keyFunc, SFunction<E, P> valueFunc, Consumer<E>... peeks);
// 查询表内记录，封装返回为Map<属性,属性>，考虑了并行流的情况
Map<A, P> map(LambdaQueryWrapper<E> wrapper, SFunction<E, A> keyFunc, SFunction<E, P> valueFunc, boolean isParallel, Consumer<E>... peeks);
```

#参数说明
类型	参数名	描述
E	entity	实体对象
A	attribute	实体属性类型,也是map中key的类型
P	attribute	实体属性类型,也是map中value的类型
LambdaQueryWrapper<E>	wrapper	支持lambda的条件构造器
SFunction<E, A>	keyFunc	封装后map中作为key的条件
SFunction<E, P>	valueFunc	封装后map中作为value的条件
boolean	isParallel	为true时底层使用并行流执行
Consumer<E>...	peeks	可叠加的后续操作

###### group
```java
// 查询表内记录，封装返回为Map<属性,List<实体>>
Map<K, List<T>> group(LambdaQueryWrapper<T> wrapper, SFunction<T, A> sFunction, Consumer<T>... peeks);
// 查询表内记录，封装返回为Map<属性,List<实体>>，考虑了并行流的情况
Map<K, List<T>> group(LambdaQueryWrapper<T> wrapper, SFunction<T, K> sFunction, boolean isParallel, Consumer<T>... peeks);
// 查询表内记录，封装返回为Map<属性,分组后对集合进行的下游收集器>
M group(LambdaQueryWrapper<T> wrapper, SFunction<T, K> sFunction, Collector<? super T, A, D> downstream, Consumer<T>... peeks);
// 查询表内记录，封装返回为Map<属性,分组后对集合进行的下游收集器>，考虑了并行流的情况
M group(LambdaQueryWrapper<T> wrapper, SFunction<T, K> sFunction, Collector<? super T, A, D> downstream, boolean isParallel, Consumer<T>... peeks);
```

#参数说明
类型	参数名	描述
T	entity	实体对象
K	attribute	实体属性类型,也是map中key的类型
D	-	下游收集器返回类型,也是map中value的类型
A	-	下游操作中间类型
M	-	最终结束返回的Map<K, D>
LambdaQueryWrapper<E>	wrapper	支持lambda的条件构造器
SFunction<E, A>	sFunction	分组依据，封装后map中作为key的条件
Collector<T, A, D>	downstream	下游收集器
boolean	isParallel	为true时底层使用并行流执行
Consumer<T>...	peeks	可叠加的后续操作

###### list
```java
// 查询表内记录，封装返回为List<属性>
List<A> list(LambdaQueryWrapper<E> wrapper, SFunction<E, A> sFunction, Consumer<E>... peeks);
// 查询表内记录，封装返回为List<属性>，考虑了并行流的情况
List<A> list(LambdaQueryWrapper<E> wrapper, SFunction<E, A> sFunction, boolean isParallel, Consumer<E>... peeks);
```

#参数说明
类型	参数名	描述
E	entity	实体对象
A	attribute	实体属性类型,也是list中元素的类型
LambdaQueryWrapper<E>	wrapper	支持lambda的条件构造器
SFunction<E, A>	sFunction	封装后list中的元素
boolean	isParallel	为true时底层使用并行流执行
Consumer<E>...	peeks	可叠加的后续操作

#### Db类
说明:

使用静态调用的方式，执行CRUD方法，避免Spring环境下Service循环注入、简洁代码，提升效率
需要项目中已注入对应实体的BaseMapper
完整使用方式见: 测试用例(opens new window)
对于参数为Wrapper的，需要在Wrapper中传入Entity或者EntityClass供寻找对应的Mapper
不建议在循环中调用，如果是批量保存，建议将数据构造好后使用 Db.saveBatch(数据) 保存
例如：

```java
// 根据id查询
List<Entity> list = Db.listByIds(Arrays.asList(1L, 2L), Entity.class);
// 根据条件构造器查询
List<Entity> list = Db.list(Wrappers.lambdaQuery(Entity.class));
// 批量根据id更新
boolean isSuccess = Db.updateBatchById(list);
```



```java
查询：
listByMap是等于，没有条件业没有模糊匹配

保存：
保存没有id时，id会自动生成

更新：
//更新某个字段 // 如userService.update(Wrappers.lambdaUpdate(User.class).set(User::getName, "张三三").eq(User::getId, "1"))
boolean update(Wrapper<T> updateWrapper);
// 更新某个字段 updateEntity中设置不需要更新的字段为空即可 如 userService.update(new User("老王", 26), Wrappers.lambdaQuery(User.class).eq(User::getId, "2"))
boolean update(T updateEntity, Wrapper<T> whereWrapper); updateEntity为null，可以在where

page：
page目前是获取全部的数据，并没有进行分页
```





