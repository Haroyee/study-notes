# 基于Springboot+Vue的博客论坛

## 一、可能用到的技术及软件

软件：IntelliJ IDEA 2021.1 x64、Sqlyog

技术：Springboot、MVC、Mybatis、Node.js 、Vue、MySql

## 二、可能要实现的功能

1、登陆、注册：分为用户登陆和管理员登陆，做到用户和管理员权限分离，同时实现不登录也可以访问页面与查看文章，但无法进行其他操作，比如评论，发博客等等。

2、用户个人页面：头像、昵称、签名、留言、ip显示、关注、关注更新、查看粉丝、其他主流媒体账号跳转、发布文章，文章管理管理、更换博客背景等等。

3、网页主页面：实现搜索，文章查看

4、管理员页面：管理用户发布的文章，对用户进行封禁，查看网站流量分析

## 三、数据库设计

1、Article

| 名称             | 类型     | SQL语句                                                      | 描述                               |
| ---------------- | -------- | ------------------------------------------------------------ | ---------------------------------- |
| article_id       | int      | int AUTO_INCREMENT                                           | 文章id                             |
| article_title    | varchar  | varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL | 文章标题                           |
| article_cover    | varchar  | varchar(1024) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL | 文章封面地址                       |
| author_id        | int      | int NOT NULL                                                 | 创建者id                           |
| article_pageview | int      | int NOT NULL DEFAULT 0                                       | 浏览量                             |
| article_content  | longtext | CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL    | 文章内容                           |
| article_tag      | int      | int NOT NULL                                                 | 文章分类                           |
| article_type     | tinyint  | tinyint(1) NOT NULL DEFAULT 0                                | 文章类型（0 原创 1转载 ）          |
| article_statue   | tinyint  | tinyint(1) NOT NULL DEFAULT 0                                | 文章状态（0 草稿 1待审核 2已发布） |
| is_comment       | tinyint  | tinyint(1) NOT NULL DEFAULT 1                                | 是否可评论（0否 1是）              |
| is_top           | tinyint  | tinyint(1) NOT NULL DEFAULT 0                                | 是否顶置（0 否 1是）               |
| is_pick          | tinyint  | tinyint(1) NOT NULL DEFAULT 0                                | 是否加精（0 否 1是）               |
| is_detele        | tinyint  | tinyint(1) NOT NULL DEFAULT 1                                | 是否删除 （0否 1是）               |
| create_time      | datetime | datetime NOT NULL                                            | 创建时间                           |
| update_time      | datetime | datetime NULL DEFAULT NULL                                   | 更新时间                           |

2、

## 四、权限分配

1、普通用户

浏览未被删除文章

浏览非草稿文章

评论

## 五、接口设计

后台：

1、LoginController

（1）登陆

接收：用户名与密码

返回：token

（2）注册

接收：用户名与密码

返回：null

（3）登出

接收：token

返回：null

2、userController

（1）分页查询

接受：分页查询条件+token

```java
/**
*用户查询条件
*/
public class UserQuery extends PageQuery{
    /**
     * 用户id
     */
    private Integer id;
    /**
     *  用户名(用于登陆)
     */
    private String username;
    /**
     * 用户状态（0：离线 1：在线 2：封禁）
     */
    private Integer statue;
    /**
     * 逻辑删除（0否1是）
     */
    private Boolean isDel;
    /**
     * 是否封禁
     */
    private Boolean isBan;
    /**
     * 注册时间：开始
     */
    private String registerTimeStart;
    /**
     * 注册时间：结束
     */
    private String registerTimeEnd;
    /**
     * 登陆时间：开始
     */
    private String loginTimeStart;
    /**
     * 登陆时间：结束
     */
    private String loginTimeEnd;

}
```

返回：用户表

```java
public class UserResponse {
    /**
     * 用户id
     */
    private Integer id;
    /**
     * 用户名
     */
    private String username;
    /**
     * 用户类型
     */
    private Integer userType;
    /**
     * 用户状态
     */
    private Integer statue;
    /**
     * 逻辑删除
     */
    private Boolean isDel;
    /**
     * 是否封禁
     */
    private Boolean isBan;
    /**
     * 注册时间
     */
    private LocalDateTime registerTime;
    /**
     * 登陆时间
     */
    private LocalDateTime loginTime;
}

```

（2）新增用户

接收：用户名+密码+token

返回：null

（3）修改用户封禁状态

接收：id+封禁状态字isBan +token

返回：null

（4）删除用户

接受：List<Integer>:用户id数组+token

返回：null

（5）更新用户信息

接收：用户名与密码+token

返回：null

（6）下线

接受：id+token

返回：null

2、userInfoController

（1）分页查询

接收：分页查询条件+token

返回：整个userInfo

（2）更新

接收：