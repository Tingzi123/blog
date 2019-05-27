title: SpringBoot与缓存
---
缓存的场景

临时性数据存储【校验码】
避免频繁因为相同的内容查询数据库【查询的信息】
1、JSR107缓存规范

用的比较少
Java Caching定义了5个核心接口

1）CachingProvider

定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期间访问多个CachingProvider

2）CacheManager

定义了创建、配置、获取、管理和控制多个唯一命名的Cache,这些Cache存在于CacheManage的上下文中，一个CacheManage只被一个CachingProvider拥有

3）Cache

类似于Map的数据结构并临时储存以key为索引的值，一个Cache仅仅被一个CacheManage所拥有

4）Entry

存储在Cache中的key-value对

5）Expiry

存储在Cache的条目有一个定义的有效期，一旦超过这个时间，就会设置过期的状态，过期无法被访问，更新，删除。缓存的有效期可以通过ExpiryPolicy设置。
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah1.png?raw=true)

2、Spring的缓存抽象

包括一些JSR107的注解

CahceManager

Cache

1、基本概念

重要的概念&缓存注解
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah2.png?raw=true)

2、整合项目

1、新建一个SpringBoot1.5+web+mysql+mybatis+cache

2、编写配置文件，连接Mysql
```java
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/spring_cache
spring.datasource.username=root
spring.datasource.password=123456
# 开启驼峰命名匹配
mybatis.configuration.map-underscore-to-camel-case=true
# 打印sql
logging.level.com.cuzz.cache.mapper=debug
# 可以打印配置报告
debug=true
```
3、创建一个bean实例

Department
```java
package com.cuzz.cache.bean;

import java.io.Serializable;

public class Department implements Serializable {
    
    private Integer id;
    private String deptName;

    public Department(){
    }
    
	// getter/setter
}
```
Employee
```java
package com.cuzz.cache.bean;

import java.io.Serializable;

public class Employee implements Serializable {
    
    private Integer id;
    private String lastName;
    private String gender;
    private String email;
    private Integer dId;

	public Employee() {
    }
    // getter/setter
}
```
4、创建mapper接口映射数据库，并访问数据库中的数据
```java
package com.cuzz.cache.mapper;

import com.cuzz.cache.bean.Employee;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;

/**
 * @Author: cuzz
 * @Date: 2018/9/26 15:54
 * @Description:
 */
@Mapper
public interface EmployeeMapper {

    @Select("SELECT * FROM employee WHERE id = #{id}")
    Employee getEmployeeById(Integer id);

    @Update("UPDATE employee SET last_name=#{lastName},email=#{email},gender=#{gender},d_id=#{dId} WHERE id=#{id}")
    void updateEmp(Employee employee);

    @Delete("DELETE FROM employee WHERE employee.id=#{id}")
    void deleteEmp(Integer id);

    @Select("SELECT * FROM employee WHERE last_name=#{lastName}")
    Employee getEmpByLastName(String lastName);

}
```
5、主程序添加注解MapperScan，并且使用@EnableCaching开启缓存
```java
package com.cuzz.cache;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@MapperScan("com.cuzz.cache.mapper")
@SpringBootApplication
@EnableCaching
public class Springboot07CacheApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot07CacheApplication.class, args);
	}
}
```
6、编写service，来具体实现mapper中的方法

将方法的运行结果进行缓存，以后要是再有相同的数据，直接从缓存中获取，不用调用方法

CacheManager中管理多个Cache组件，对缓存的真正CRUD操作在Cache组件中，每个缓存组件都有自己的唯一名字；

属性：
```java
CacheName/value:指定存储缓存组件的名字
key:缓存数据使用的key,可以使用它来指定。默认是使用方法参数的值，1-方法的返回值
编写Spel表达式：#id 参数id的值， #a0/#p0 #root.args[0]
keyGenerator:key的生成器，自己可以指定key的生成器的组件id
key/keyGendertor二选一使用
cacheManager指定Cache管理器，或者cacheReslover指定获取解析器
condition:指定符合条件的情况下，才缓存；
unless：否定缓存，unless指定的条件为true，方法的返回值就不会被缓存，可以获取到结果进行判断
sync:是否使用异步模式，unless不支持
```

```java
package com.cuzz.cache.service;

import com.cuzz.cache.bean.Employee;
import com.cuzz.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;


/**
 * @Author: cuzz
 * @Date: 2018/9/26 16:08
 * @Description:
 */
@Service
public class EmployeeService {

    @Autowired
    EmployeeMapper employeeMapper;

    @Cacheable(cacheNames = "emp")
    public Employee getEmployee(Integer id) {
        System.out.println("----> 查询" + id + "号员工");
        return employeeMapper.getEmployeeById(id);
    }
}
```
7、编写controller测试
```java
@RestController
public class EmployeeController {
    @Autowired
    EmployeeService employeeService;

    @GetMapping("/emp/{id}")
    public Employee getEmp(@PathVariable("id")Integer id){
        return employeeService.getEmp(id);
    }
}
```
8、测试结果

第一次访问会查询数据库，之后再次查询直接访问缓存中取值

3、缓存原理

1、原理：

1、CacheAutoConfiguration

2、导入缓存组件
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah3.png?raw=true)

3、查看哪个缓存配置生效
```java
SimpleCacheConfiguration生效
```

4、给容器注册一个CacheManager:ConcurrentMapCacheManager

5、可以获取和创建ConcurrentMapCache,作用是将数据保存在ConcurrentMap中

2、运行流程

1、方法运行之前，先查Cache(缓存组件），按照cacheName的指定名字获取；

​	（CacheManager先获取相应的缓存），第一次获取缓存如果没有cache组件会自己创建

2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；

​	key是按照某种策略生成的，默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key

​	没有参数 key=new SimpleKey()

​	如果有一个参数 key=参数值

​	如果多个参数 key=new SimpleKey(params);

3、没有查到缓存就调用目标方法

4、将目标方法返回的结果，放回缓存中

​	方法执行之前，@Cacheable先来检查缓存中是否有数据，按照参数的值作为key去查询缓存，如果没有，就运行方法，存入缓存，如果有数据，就取出map的值。

3、自定义缓冲KeyGenerator
```java
@Configuration
public class MyCacheConfig {
    
    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object o, Method method, Object... objects) {
                return method.getName() + "[" + Arrays.asList(objects) + "]";
            }
        };
    }
}
```
指定keyGenerator
```java
@Service
public class EmployeeService {

    @Autowired
    EmployeeMapper employeeMapper;

    @Cacheable(cacheNames = "emp", keyGenerator = "myKeyGenerator")
    public Employee getEmployee(Integer id) {
        System.out.println("----> 查询" + id + "号员工");
        return employeeMapper.getEmployeeById(id);
    }
}
```
4、Cache的注解

1、@Cacheput

修改数据库的某个数据，同时更新缓存

运行时机

先运行方法，再将目标结果缓存起来

1、编写更新方法
```java
@CachePut(value = {"emp"})
public Employee updateEmployee(Employee employee) {
    System.out.println("---->updateEmployee"+employee);
    employeeMapper.updateEmp(employee);
    return employee;
}
```
2、编写Controller方法
```java
@GetMapping("/emp")
public Employee update(Employee employee) {
    return employeeService.updateEmployee(employee);
}
```
测试

测试步骤

1、先查询1号员工

2、更新1号员工数据

3、查询1号员工

可能并没有更新，

是因为查询和更新的key不同

cacheable的key是不能使用result的参数的,因为他在方法之前执行，还没有生成的结果作为result的参数

所以要使@CachePut使用@Cacheable使用相同的key
```java
@CachePut(value = {"emp"}, key = "#result.id")
public Employee updateEmployee(Employee employee) {
    System.out.println("---->updateEmployee"+employee);
    employeeMapper.updateEmp(employee);
    return employee;
}
```
前面用的是Id 后面用的是Employee

效果：

第一次查询：查询mysql
第二次更新：更新mysql
第三次查询：调用内存

2、CacheEvict

清除缓存

编写测试方法

service
```java
@CacheEvict(value = "emp",key = "#id")
public  void  deleteEmployee(Integer id){
    System.out.println("---->删除的employee的id是: "+id);
}
```
controller
```java
@GetMapping("/delete")
public String delete(Integer id) {
    employeeService.deleteEmployee(id);
    return "success";
}
```
allEntries = true，代表不论清除那个key，都重新刷新缓存

beforeInvocation=true 方法执行前，清空缓存，默认是false,如果程序异常，就不会清除缓存

3、Caching定义组合复杂注解

组合

Cacheable
CachePut
CacheEvict
CacheConfig抽取缓存的公共配置
```java
@Caching(
        cacheable = {
            @Cacheable(value = "emp",key = "#lastName")
        },
        put = {
            @CachePut(value = "emp",key = "#result.id"),
            @CachePut(value = "emp",key = "#result.gender")
        }
    )
    public Employee getEmployeeByLastName(String lastName) {
        return employeeMapper.getEmpByLastName(lastName);
    }
```
如果查完lastName，再查的id是刚才的值，就会直接从缓存中获取数据

4、CacheConfig抽取缓存的公共配置
```java
@CacheConfig(cacheNames = "emp")
@Service
public class EmployeeService {
```
然后下面的value=emp就不用写了

5、Redis

默认的缓存是在内存中定义HashMap，生产中使用Redis的缓存中间件

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件

1、安装redis在docker上
```java
#拉取redis镜像
docker pull registry.docker-cn.com/library/redis
#启动redis[e1a73233e3be]docker images的id
 docker run -d -p 6379:6379 --name redis01 e1a73233e3be
```
2、Redis的Template

Redis的常用五大数据类型

String【字符串】、List【列表】、Set【集合】、Hash【散列】、ZSet【有序集合】

分为两种一种是StringRedisTemplate（redis操作对象大都是字符串，为简化字符串操作特地抽取出来的类，操作k-v对象都是字符串的），另一种是RedisTemplate（操作k-v都是对象的）

根据不同的数据类型，大致的操作也分为这5种，以StringRedisTemplate为例
```java
stringRedisTemplate.opsForValue()  --String
stringRedisTemplate.opsForList()  --List
stringRedisTemplate.opsForSet()  --Set
stringRedisTemplate.opsForHash()  --Hash
stringRedisTemplate.opsForZset()  -Zset
```
1、导入依赖
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
2、修改配置文件
```java
spring.redis.host=47.103.24.194
```
3、添加测试类
```java
@Autowired
    StringRedisTemplate stringRedisTemplate;//操作字符串【常用】

    @Autowired
    RedisTemplate redisTemplate;//操作k-v都是对象    
	@Test
    public void test01(){
        // stringRedisTemplate.opsForValue().append("msg", "hello");
        String msg = stringRedisTemplate.opsForValue().get("msg");
        System.out.println(msg);
    }
```
写入数据
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah4.png?raw=true)

3、测试保存对象

对象需要序列化
1、序列化bean对象
```java
public class Employee implements Serializable {
```
2、将对象存储到Redis
```java
@Test
public  void test02(){
    Employee emp = employeeMapper.getEmpById(2);
    redisTemplate.opsForValue().set("emp-01", emp);
}
```
3、结果
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah6.png?raw=true)

4、以json方式传输对象

1、新建一个Redis的配置类MyRedisConfig,
```java
/**
 * @Author: cuzz
 * @Date: 2018/9/26 23:50
 * @Description:
 */
@Configuration
public class MyRedisConfig {
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> jsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(jsonRedisSerializer);
        return template;
    }
}
```
2、编写测试类
```java
 @Autowired
 RedisTemplate<Object,Employee> empRedisTemplate;
@Test
public  void test02(){
    Employee emp = employeeMapper.getEmpById(2);
    empRedisTemplate.opsForValue().set("emp-01", emp);

}
```
3、测试效果
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah5.png?raw=true)

5、整合redis作为缓存

使用缓存时，默认使用的是ConcurrentMapCache，将数据保存在ConcurrentMap中，开发中使用的是缓存中间件，redis、memcached、ehcache等

starter启动时，有顺序，redis优先级比ConcurrentMapCache更高，CacheManager变为RedisCacheManager，所以使用的是redis缓存

传入的是RedisTemplate<Object, Object>

默认使用的是jdk的序列化保存
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah7.png?raw=true)
我们可以自定义CacheManager

编写一个配置类
```java
@Configuration
public class MyRedisConfig {
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> jsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(jsonRedisSerializer);
        return template;
    }

    @Bean
    public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        // 使用前缀，默认将CacheName作为前缀
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }
}
```
结果变为json格式
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootcah8.png?raw=true)
empRedisTemplate只能用来反序列化Employee，而不能反序列化Department对象，因此要创建不同的xxxRedisTemplate

分别创建Manager
```java
@Configuration
public class MyRedisConfig {
    // Employee
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> jsonRedisSerializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(jsonRedisSerializer);
        return template;
    }
    // Employee
    @Primary  // 有多个cacheManager需要制定一个默认的一般将jdk的cacheManager作为默认
    @Bean
    public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }

    // Department
    @Bean
    public RedisTemplate<Object, Department> deptRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate<Object, Department>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> jsonRedisSerializer = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(jsonRedisSerializer);
        return template;
    }
    
    // Department
    @Bean
    public RedisCacheManager departmentCacheManager(RedisTemplate<Object, Department> deptRedisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }

}
```
指定CacheManager
```java
@CacheConfig(cacheNames = "emp", cacheManager = "employeeCacheManager")
@Service
public class EmployeeService {}
```