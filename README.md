# 冬令营之从入门到拿奖

## 准备

- tomcat（确保已经安装）
- idea（要专业版的，社区版没有tomcat的部署选项，需要卸载重装）
- api链接: https://apifox.com/apidoc/shared-2420b892-1aab-4144-83c8-c16432758ba5 访问密码: quanta2024
- postman或apifox
- 前端页面（dist.zip）

## 前提知识

- **token？**

  HTTP无状态协议：对于业务处理没有记忆能力，之前做了啥完全记不住。每次请求都是完全独立互不影响的，没有任何上下文信息。

  假如一直用这种原生无状态的 HTTP 协议，我们每换一个页面可能就得重新登录一次，那还玩个球。

  服务器每天接受几千万的请求，他是怎么知道某个请求是某个客户的呢？

  所以有了Seesion，Cookie和Token三种解决方法。

  ==这里主要说说Token==

  标准的Token交互：

  ![微信图片_20221110153801](http://home.linwine.space:7900/imagebed/note/微信图片_20221110153801.png)

  

  **简单点说**

  用户登陆成功后会给客户端发送一个令牌（本质是个字符串），然后客户端每次的请求（看主页，搜索）的请求上要带上这个令牌（一般放在请求头里），服务器通过解析这个令牌，就知道这个请求是由哪个客户发起的，才能继续下面的操作。

  ==冬令营注意==

  一般用户登录的时候，这个用户的Token就会刷新，来避免多个用户使用这个账号（不刷新的话，老token也能用嘛）

  有能力的小伙伴可以去实现toke过期机制（加分）：

  效果：有些网站，你登陆后一两天内不需要登录，但是过了一段时间就需要你重新登录了

  生成token的时候记录生成时间，有请求带token传入的时候，token存在的话，计算这个token生成的时间和现在的时间的差是否大于有效值（自己设计），大于则需要重新登录，小于的话可以刷新这个token的时间

  退出登录时则需要删除token记录

- **postman不会用？？？**

  **简介：**

  Postman相当于一个客户端,它可以模拟用户发起的各类HTTP请求,将请求数据发送至服务端,获取对应的响应结果, 从而验证响应中的结果数据是否和预期值相匹配;并确保开发人员能够及时处理接口中的bug,进而保证产品上线之后的稳定性和安全性。

  ![image-20221110151441650](http://home.linwine.space:7900/imagebed/note/image-20221110151441650.png)

  ![image-20221110151553421](http://home.linwine.space:7900/imagebed/note/image-20221110151553421.png)

  ![image-20221110152142727](http://home.linwine.space:7900/imagebed/note/image-20221110152142727.png)

  ==传JSON请求体？==

  ![image-20221110152237064](http://home.linwine.space:7900/imagebed/note/image-20221110152237064.png)

  ==怎么将Token放在请求头？==

  ![image-20221110152518618](http://home.linwine.space:7900/imagebed/note/image-20221110152518618.png)

  ==注意的是：这里的Authorization（即Token键名）一般跟随api文档里面的，冬令营的文档token的键名都为Authorization==

- **Get请求和Post请求傻傻分不清楚？**

  **举个栗子**

  现在有个登录接口http://www.a.com/login，需要传入的参数为用户名（name）密码（password）

  **get请求是这样的**

  ![image-20221110152347602](http://home.linwine.space:7900/imagebed/note/image-20221110152347602.png)

  **post请求是这样的**

  ![image-20221110152827338](http://home.linwine.space:7900/imagebed/note/image-20221110152827338.png)

  **区别在于**

  - url的可见性：get方法传的值一般再url显示，而post方法不会

  - 数据传输上：get方法一般通过拼接url进行传递参数。post方法一般用请求体（body）传输参数

  - 速度上：一般get比post快

    ...

- **啥是跨域？**

  出于浏览器的同源策略限制。当一个请求url的==协议==、==域名==、==端口==三者之间任意一个与当前页面url不同即为跨域。

  **简单讲**，因为我们是前后端分离的，前端页面部署的地址（比如www.a.com）和前端传值给后端处理的地址不一样 （www.b.com），那么前端发送的请求给后端，因为域名不同，浏览器就会觉得不安全，会发送一个预检请求options给服务器，如果服务器未处理这个请求，就会出现跨域的问题，所以它就会阻止发送正常的业务请求。

  ![img](http://home.linwine.space:7900/imagebed/note/70e9de32598a4783acfafbf082ff3039.png)

  ![img](http://home.linwine.space:7900/imagebed/note/b0c9f8e4eff86f1277c849554f6d8d0b.png)

  这种就是经典的跨域问题。

  **怎么解决**

  在正常请求的前面，浏览器会发送一个options请求给服务器进行预检，如果后端没有响应，会触发跨域问题，我们要做的是，在我们的controller层加上一点东西，给请求头添加 Access-Control-Allow-Origin 属性。

  ```java
  public class LoginController extends HttpServlet {
  
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp){  
          // 这里以get方法举例，post也是一样的
          resp.setHeader("Access-Control-Allow-Origin", "*");
          resp.setHeader("Access-Control-Allow-Credentials", "true");
          resp.setHeader("Access-Control-Allow-Methods", "*");
          resp.setHeader("Access-Control-Max-Age", "3600");
          resp.setHeader("Access-Control-Allow-Headers", "Authorization,Origin,X-Requested-With,Content-Type,Accept," + "content-Type,origin,x-requested-with,content-type,accept,authorization,token,id,X-Custom-Header,X-Cookie,Connection,User-Agent,Cookie,*");
          resp.setHeader("Access-Control-Request-Headers", "Authorization,Origin, X-Requested-With,content-Type,Accept");
          resp.setHeader("Access-Control-Expose-Headers", "*");
      }
  
      @Override
      protected void doOptions(HttpServletRequest req, HttpServletResponse resp){
          resp.setHeader("Access-Control-Allow-Origin", "*");
          resp.setHeader("Access-Control-Allow-Credentials", "true");
          resp.setHeader("Access-Control-Allow-Methods", "*");
          resp.setHeader("Access-Control-Max-Age", "3600");
          resp.setHeader("Access-Control-Allow-Headers", "Authorization,Origin,X-Requested-With,Content-Type,Accept," + "content-Type,origin,x-requested-with,content-type,accept,authorization,token,id,X-Custom-Header,X-Cookie,Connection,User-Agent,Cookie,*");
          resp.setHeader("Access-Control-Request-Headers", "Authorization,Origin, X-Requested-With,content-Type,Accept");
          resp.setHeader("Access-Control-Expose-Headers", "*");
      }
  }
  ```

  

- ==**开发须知**==

  最好遵守第一节第二节课培训说的开发规范，养成习惯。

  类，变量，方法名最好做到见名知意，不用拼音，不要用abc（严格扣分）

  而且格式要对，什么时候大驼峰小驼峰下划线...


## 思路

- **搭建开发框架**
- **分析接口和逻辑**
- **建库建表**
- **写代码**
- **测试**

## 上手

### 1.搭建框架

- **建立项目**

  ![image-20231122140841299](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311221409158.png)

- **引入依赖 **    ==记得点maven刷新标志==

  ```xml
      <!-- junit -->
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
      </dependency>
      <!-- servlet -->
      <dependency>
        <groupId>jakarta.servlet</groupId>
        <artifactId>jakarta.servlet-api</artifactId>
        <version>5.0.0</version>
        <scope>provided</scope>
      </dependency>
      <!-- mysql数据库 -->
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.27</version>
      </dependency>
      <!-- json转换 -->
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.78</version>
      </dependency>
      <!-- commons-io -->
      <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.11.0</version>
      </dependency>
      <!-- lombok -->
      <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.24</version>
        <scope>provided</scope>
      </dependency>
  
  ```

- **创建java文件夹**（已经有了就不用创建）

  ![](http://home.linwine.space:7900/imagebed/note/2022111014451.png)

  ![](http://home.linwine.space:7900/imagebed/note/2022111014512.png)

- **分层**

  ![](http://home.linwine.space:7900/imagebed/note/20221110141017.png)

  ![](http://home.linwine.space:7900/imagebed/note/20221110141042.png)

  ![](http://home.linwine.space:7900/imagebed/note/20221110144142.png)

  - **dao/mapper层**
    DAO层叫数据访问层，全称为data access object。

    某个DAO一定是和数据库的某一张表一一对应的，其中封装了CRUD（增加Create、检索Retrieve、更新Update和删除Delete）基本操作，DAO只做原子操作。无论多么复杂的查询，dao只是封装增删改查。至于增删查改如何去实现一个功能，dao是不管的。

    impl：具体实现。

  - **service层**
    Service层叫服务层，被称为服务，粗略的理解就是对一个或多个DAO进行的再次封装，封装成一个服务，所以这里也就不会是一个原子操作了，需要事务控制。管理具体的功能的。

  - **pojo/entity层**

    对应的数据库表的实体类(如User类)。

  - **utils层**

    工具类层，包括JDBC工具什么的。

  - **controller层**

    负责解析前端参数的传入和传出，调用相关的service层
  
  ![未命名文件](http://home.linwine.space:7900/imagebed/note/未命名文件.png)
  
  ==如果建层的时候文件夹层级变成这样可以改一下这个==
  
  ![image-20221110171738406](http://home.linwine.space:7900/imagebed/note/image-20221110171738406.png)
  
  ![image-20221110171753271](http://home.linwine.space:7900/imagebed/note/image-20221110171753271.png)
  
  点一下那个齿轮标志，然后取消打勾就行了
  
- **导入工具类，配置**

  JDBC工具类，修改配置文件路径，建立数据库连接

  ![image-20221110171851360](http://home.linwine.space:7900/imagebed/note/image-20221110171851360.png)

### 2.分析接口

**以登录接口为例子**

<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox1.png" style="zoom: 70%;" />

- **请求方式**：一般有get和post方法，和servlet中的doGet和doPost对应
- **请求路径**：前端根据这个地址传值，后端要处理这个地址的请求
- **请求格式**：有很多种格式，冬令营普遍用json传值
- **响应格式**：返回数据的格式，严格遵守，否则前端无法解析

### 3.分析业务逻辑

**还是以登录接口为栗子**

登录的逻辑是什么样的？

看看传入的参数：email，password

<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox1-5.jpg" alt="image-20231122151455978" style="zoom: 67%;" />

传出的参数：token

<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox2.png" alt="image-20231122151507288" style="zoom:67%;" />

验证账户是否存在--》dao层去数据库查是否有这个账户--》service判断

验证密码是不是正确--》dao层查询这个账户的密码--》service判断

登录成功后，token怎么来的--》服务器随机生成，并且和这个账户绑定--》要存入数据库

所以我们知道了要有一张表存邮箱，用户密码和token（当然这只是这张表的一部分）

token机制看前提知识里的token和冬令营须知

### 4.建表

还记得建表三必备字段吗？

建个用户表

分析接口的时候我们知道了要有email，password和token

想想用户表还有什么是必要的字段？用户名？也加上去

![image-20231122205329206](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222053237.png)

id一般自增，且无符号

时间戳设置初始化，自动更新

字段哪些唯一，哪些要设置索引

...

这些就不一一演示了

### 5.开写java

**以登录接口为栗子**

先配置tomcat统一处理路径。

<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox3.png" alt="image-20231122151825600" style="zoom: 80%;" />



<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox4.png" alt="image-20231122151809766" style="zoom: 80%;" />



<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox5.png" alt="image-20231122152752307" style="zoom:80%;" />

查看文档里面的所有的请求路径都有一个共同的前缀路径 **http://8.138.110.206:9090**

**配置一下tomcat和统一路径**

右上角点一下当前文件--编辑配置

![image-20221110174020645](http://home.linwine.space:7900/imagebed/note/image-20221110174020645.png)

添加一个本地tomcat服务器（没有这个选项的话，要重装idea为标准版，这个第一节课已经讲了，去Q群拿安装教程）

![image-20221110174104095](http://home.linwine.space:7900/imagebed/note/image-20221110174104095.png)



端口号默认8080，冬令营是用9090，修改成9090

每次开启tomcat服务器会弹出浏览器窗口，觉得烦的话可以关闭”在启动后“前面的勾

![image-20231122200253569](https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/202311222002619.png)

点部署，设置一下工件和应用程序上下文(项目名)为统一路径

但在本次冬令营的项目没有明确的项目名，同学们可以自己设置一个

![image-20231122200828887](https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/202311222008935.png)

确认，运行测试一下

![image-20231122200938805](https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/Weixin%20Screenshot_20241104232531.png)

工件已成功部署就行了。



#### **pojo层**

有什么表，就建什么实体类。属性一一对应

数据库实现的时间戳自动更新，类里面就可以不带了

![image-20231122205345422](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222053459.png)

------

#### controller层

新建一个controller类来处理这个请求地址。

![image-20221110171940182](http://home.linwine.space:7900/imagebed/note/image-20221110171940182-16680720142633.png)

==记得类继承HttpServlet和进行跨域处理，乱码处理==

![image-20231122202355466](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222023536.png)

==写上处理的请求路径==

![image-20231122202252103](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222022138.png)

name给这个servlet接口取名，地址填上api文档除了统一路径后面的东西

当然也可以通过web.xml配置，上节课有讲，不再赘述

**开始传入的取值**

这个接口传入的参数只有请求体里面的json

![image-20231122202928507](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222029543.png)

![image-20231122214440349](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222144443.png)

注意是org.apache.commons.io.IOUtils

==其他方式的取值==

- get方法

  <img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox6.png" alt="image-20231122203200468" style="zoom:67%;" />

  我们可以使用这种方法去获得它的请求参数

  ```java
  req.getParameter("pageNum");
  req.getParameter("pageSize");
  req.getParameter("pagemodule");
  ```

  请求头里获得token
  
   ```java
   req.getHeader("Authorization");
   ```

==controller写到这里需要调用service，我们去写service层==



------

#### service层

写一个service类，写上controller要调用的方法，注意下放回的类型和传入参数类型。

![image-20231122203938163](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222039203.png)

service要判断账户在不在是不是要去dao层拿？

那就得写dao层了



------

#### dao层

由上面的service我们知道dao层要提供一个能根据email返回密码的方法和一个插入用户token的方法。

我这里写个根据email返回整个user信息的接口，根据id更新token的方法，方便其他service去调用。

现在dao创建一个userDao接口，提供getUserByPhone，getUserByUsername，updateTokenById等方法。

![image-20231122204458217](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222044268.png)

现在需要一个实现类去实现上面的接口

在dao.impl创建一个类叫UserDaoImpl，去实现接口里的方法

![image-20231122204332515](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222043568.png)

**开始写具体实现**

![image-20231122205654798](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222056867.png)

![image-20221110190034568](http://home.linwine.space:7900/imagebed/note/image-20221110190034568.png)



------

#### 回到service层

调用dao层，写业务逻辑。

==注意的是一般出现逻辑错误，比如用户不存在...，返回前端的json一般都是code状态码（400），外加msg信息==

![image-20231122210914294](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222109394.png)



------

#### 回到controller层

调用service层

![image-20231122211158369](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222111437.png)

==写完记得重新部署tomcat==



------

#### 测试

先给表增加点数据

![image-20231122211316155](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222113183.png)

用postman测试

<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/Weixin%20Screenshot_20241106195036.png" alt="image-20231122212028157" style="zoom:50%;" />

<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox10.png" alt="image-20231122212018121" style="zoom:50%;" />



<img src="https://legalsys.oss-cn-guangzhou.aliyuncs.com/myxy/apifox11.png" alt="image-20231122212007224" style="zoom: 50%;" />

![image-20231122212243103](https://typora-1314255022.cos.ap-guangzhou.myqcloud.com/typora/202311222122133.png)

数据库也更新了。

自此后端的数据是没问题的。至于前端如何看到效果，那就是后话了 。

## 获奖宝典

- 首先还是接口的实现情况，尽量写吧，不一定要完全写完。没有时间的情况下还是以学业为主，冬令营次要。
- 写其他接口要先考虑逻辑，比如注册的时候邮箱用不用判断是否存在，姓名能不能重复
- 代码规范性，命名，注释也很重要
- 代码封装程度，其实很多东西可以封装起来，跨域处理，编码设置，token的生成... 封装后不用再去复制粘贴，代码也不会一坨坨的，何乐而不为
- 有一些接口实现起来可能比较难，比如分页，发邮件，保存文件。可以先从简单的做起，难的有兴趣有时间可以去折腾，早晚要学，实现了当然加分
- 冬令营展示时一定要把自己写的东西展示给我们看，有优势有亮点可以说一说。看漏看少付出了没回报岂不是很亏...
- 名次不重要，重要的是在开发过程中面对困难的解决方法和宝贵的经验，这些都比名次重要得多
- 遇到问题首先要自己思考，看不懂英文可以翻译，大多时候百度csdn比学长学姐讲得还细，实在没头绪可以请假学长学姐。

# 预祝大家冬令营取得好成果🎉

