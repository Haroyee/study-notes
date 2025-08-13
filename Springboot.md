# Springboot

## 一、配置

1、常用依赖

```xml
<!--springboot父依赖-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.13</version>
</parent>
<!-- web开发 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

 <!-- mysql驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
<!-- druid 数据库连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.16</version>
        </dependency>
<!-- mybatis plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.2</version>
        </dependency>
<!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
<!--引入Knife4j的官方start包,该指南选择Spring Boot版本<3.0,开发者需要注意-->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-openapi2-spring-boot-starter</artifactId>
            <version>4.4.0</version>
        </dependency>
<!-- redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
 <!-- Redis连接池 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
<!-- spring websocket -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
<!--Springboot-maven打包插件-->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.6.13</version>          
</plugin>
```

2、yml配置

```yml
server:
  port: 8080
# mybatisPlus配置
mybatis-plus:
  # mapper映射地址
  mapper-locations: classpath:mapper/*.xml
  # 实体类扫描包路径
  type-aliases-package: com.har.entity
  configuration:
    #    # sql打印
    #    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    # 开启驼峰命名
    map-underscore-to-camel-case: true
  global-config:
    db-config:
      #数据库自增策略
      id-type: auto
      # 数据库表前缀
      table-prefix: t_


# 连接数据库的信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    # ip,端口,库名
    url: jdbc:mysql://localhost:3306/myblog?serverTimezone=UTC&&useSSL=false
    # 用户名和密码
    username: root
    password: root
    #数据库连接池-阿里巴巴
    type: com.alibaba.druid.pool.DruidDataSource
#自定义全局参数

```

3、spring 表单验证

| 源码                      | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| @Null                     | 限制只能为null                                               |
| @NotNull                  | 限制必须不为null                                             |
| @NotEmpty                 | 验证注解的元素值不为null且不为空(字符串长度不为0、集合大小不为0） |
| @NotBlank                 | 验证注解的元素值不为空(不为null、去除首位空格后长度为0〕，不同于@NotEmpty,@NotBlank只应用于字符串且在比较时会去除字符串的空格 |
| @AssertFalse              | 限制必须为false                                              |
| @AssertTrue               | 限制必须为true                                               |
| @size(max,min)            | 限制字符长度必须在min到max之间                               |
| @Past                     | 限制必须是一个过去的日期                                     |
| @Future                   | 验证日期为当前时间之后                                       |
| @PastOrPresent            | 制日期为当前时间或之前                                       |
| @FutureOrPresent          | 验证日期为当前时间或之后                                     |
| @Max(value)               | 限制必须为一个不大于指定值的数字                             |
| @Min(value)               | 限制必须为一个不小于指定值的数字                             |
| @DecimalMin(value)        | 限制必须为一个不小于指定值的数字                             |
| @DecimalMax(value)        | 限制必须为一个不大于指定值的数字                             |
| @Digits(integer,fraction) | 限制必须为一个小数，目整数部分的位数不能超过integer，小数部分的位数不能超过fraction |
| @Negative                 | 限制必须为负整数                                             |
| @NegativeOrZero           | 限制必须为负整数或零                                         |
| @Positive                 | 限制必须为正整数                                             |
| @PositiveOrZero           | 限制必须为正整数或零                                         |
| Pattern(value)            | 限制必须符合指定的正则表达式                                 |
| @Email                    | 限制必须为email格式                                          |

## 二、依赖学习

1、Spring Security 

导入

```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
        <version>2.6.13</version>
</dependency>
```

认识Security结构

debug

```
run().getBean(DefaultSecurityFilterChain.class)
```

得到结构（过滤器共15层）

![image-20240408155332466](C:\Users\CXQ\AppData\Roaming\Typora\typora-user-images\image-20240408155332466.png)

<img src="C:\Users\CXQ\AppData\Roaming\Typora\typora-user-images\image-20240408160537187.png" alt="image-20240408160537187" style="zoom:200%;" />

WebAsyncManagerIntegrationFilter：从页面获取用户名和密码

重写

(1)Security UserDetails层 实现 返回数据库中的User信息

```java
/**
 * Security UserDetails 实现 返回数据库中的User信息
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class SecurityUser implements UserDetails {
    private User user;
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getPassword();
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
```

(2)Security UserDetailService层 实现

```java
/**
 * Security UserDetailService 实现
 */
@Service("UserDetailsService")
public class SecurityServiceImpl implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(User::getUsername,username);
        User user = userMapper.selectOne(queryWrapper);
        if (user == null) System.out.println("error");
        else System.out.println("success");
        return new SecurityUser(user);
    }
}
```

(3)SecurityConfig

2.5.7版本前

三、工具类

redis

```java
@Component
public class RedisUtil {
    /**
     * 如果使用 @Autowired 注解完成自动装配 那么
     * RedisTemplate要么不指定泛型,要么泛型 为<Stirng,String> 或者<Object,Object>
     * 如果你使用其他类型的 比如RedisTemplate<String,Object>
     * 那么请使用 @Resource 注解
     * */
    @Resource
    private RedisTemplate<String,Object> redisTemplate;

    private Logger logger = LoggerFactory.getLogger(this.getClass());
    /**
     * 缓存value
     *
     * @param key -
     * @param value -
     * @param time -
     * @return -
     */
    public boolean cacheValue(String key, Object value, long time) {
        try {
            ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
            valueOperations.set(key, value);
            if (time > 0) {
                // 如果有设置超时时间,经过time秒后删除键值
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Throwable e) {
            logger.error("缓存[" + key + "]失败, value[" + value + "] " + e.getMessage());
        }
        return false;
    }

    /**
     * 缓存value，没有设置超时时间
     *
     * @param key -
     * @param value -
     * @return -
     */
    public boolean cacheValue(String key, Object value) {
        return cacheValue(key, value, -1);
    }

    /**
     * 判断缓存是否存在
     *
     * @param key
     * @return
     */
    public boolean containsKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Throwable e) {
            logger.error("判断缓存是否存在时失败key[" + key + "]", "err[" + e.getMessage() + "]");
        }
        return false;
    }

    /**
     * 根据key，获取缓存
     *
     * @param key -
     * @return -
     */
    public Object getValue(String key) {
        try {
            ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
            return valueOperations.get(key);
        } catch (Throwable e) {
            logger.error("获取缓存时失败key[" + key + "]", "err[" + e.getMessage() + "]");
        }
        return null;
    }

    /**
     * 移除缓存
     *
     * @param key -
     * @return -
     */
    public boolean removeValue(String key) {
        try {
            redisTemplate.delete(key);
            return true;
        } catch (Throwable e) {
            logger.error("移除缓存时失败key[" + key + "]", "err[" + e.getMessage() + "]");
        }
        return false;
    }

    /**
     * 根据前缀移除所有以传入前缀开头的key-value
     *
     * @param pattern -
     * @return -
     */
    public boolean removeKeys(String pattern) {
        try {
            Set<String> keySet = redisTemplate.keys(pattern + "*");
            redisTemplate.delete(keySet);
            return true;
        } catch (Throwable e) {
            logger.error("移除key[" + pattern + "]前缀的缓存时失败", "err[" + e.getMessage() + "]");
        }
        return false;
    }
}
```

jwt

```java
package com.har.utils;

import com.har.entity.User;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;
@Component
public class JwtUtil {
    // 这里的密钥SECRET是用于加密的密钥，应该保密且足够复杂
    private  static String SECRET;
    //token 过期时间
    private   static  long TIME_OUT;

    @Value("${custom_config.jwt.secret}")
    public void setSECRET(String secret){
        SECRET = secret;
    }
    @Value("${custom_config.jwt.timeout}")
    public void setTimeOut(long timeOut){
        TIME_OUT = timeOut;
    }
    // 生成JWT Token的方法
    public static String generateToken(Object object) {

        Map<String, Object> claims = new HashMap<>();

// 使用反射来获取对象的所有字段和相应的值
        for (java.lang.reflect.Field field : object.getClass().getDeclaredFields()) {
            field.setAccessible(true); // 允许访问私有字段
            try {
// 将字段名和字段值放入claims
                claims.put(field.getName(), field.get(object));
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }

        return Jwts.builder()
                .setClaims(claims) // 设置声明
                .setSubject("user") // 设置主题
                .setIssuedAt(new Date(System.currentTimeMillis())) // 设置签发时间
                .setExpiration(new Date(System.currentTimeMillis() + TIME_OUT * 1000)) // 设置过期时间
                .signWith(SignatureAlgorithm.HS512, SECRET) // 设置签名使用的签名算法和密钥
                .compact(); // 构建JWT并将其压缩为一个安全的URL字符串
    }
    //设置解密Token
    public static Claims decodeToken(String token){
        return Jwts.parser()
                .setSigningKey(SECRET)
                .parseClaimsJws(token)
                .getBody();
    }
}
```

web

```java
package com.har.utils;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
public class WebUtil
{
    /**
     * 将字符串渲染到客户端
     * *
     @param response 渲染对象
      * @param string 待渲染的字符串
     * @return null
     */
    public static String renderString(HttpServletResponse response, String
            string) {
        try
        {
            response.setStatus(200);
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
            response.getWriter().print(string);
        } catch (IOException e)
        {
            e.printStackTrace();
        } return null;
    }
}
```
