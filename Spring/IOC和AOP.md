

## IOC控制反转

IOC（Inversion of control）不是技术，是一种编程思想。ioc意味着将设计好的对象交给容器控制，而不是传统的在对象内部直接控制。也可以成为依赖倒置。（补充：不倒置就是A主动去获取B的实例，导倒置就是由其他人自动将B送上门来）

`控制 ：指的是对象创建(实例化、管理)的权力`

`反转 ：控制权交给外部环境(Spring 框架、IoC 容器)`

> IoC  容器实际上就是个 Map(key，value),Map 中存放的是各种对象。

- **‘谁控制了谁 控制什么 为什么是反转 哪些方面反转了’**

　　　　　　　　谁控制谁：IOC容器控制了对象

　　　　　　　　控制什么：控制了外部资源获取

　　　　　　　　为什么是反转：IOC容器帮助查找及注入依赖对象，对象只是被动的接受依赖对象

　　　　　　　　哪些方面反转：依赖对象获取被反转

`IoC 最常见以及最合理的实现方式叫做依赖注入(Dependency Injection，简称 DI)。`

- **DI（依赖注入）：“谁依赖谁 为什么需要依赖 谁注入谁 注入了什么”**

　　　　　　　　谁依赖谁：应用程序依赖于ioc容器

　　　　　　　　为什么需要依赖：应用程序需要ioc容器来提供对象需要的外部资源

　　　　　　　　谁注入谁：Ioc容器注入应用程序某个对象，应用程序依赖的对象

　　　　　　　　注入了什么：注入某个对象所需要的外部资源

`依赖注入方式：手动装配 和 自动装配`

- `手动装配：一般进行配置信息都采用手动`  `基于xml装配：构造方法、setter方法`
- `自动装配（基于注解装配）：` `属性注入指创建对象时，向类对象的属性设置属性值。` 

　 `在Spring框架中支持set方法注入和有参构造函数注入，即创建对象后通过set方法设置属性或采用有参构造函数创建对象并初始化属性。`

> 例如：
>
> 现有A依赖于B
>
> 传统开发方式：在类A中手动new一个B的对象
>
> IoC:不通过new来创建对象，而是通过IoC容器（Spring框架）来帮助我们实例化对象。需要哪个对象直接从IoC容器里面获取即可。
>
> 对比：我们“丧失了一个权力”（创建、管理对象的权力），从而也得到一个好处（不用再考虑对象的创建、管理等一系列事情）
>
> 假设A是UserAction类，而B是UserService类
>
> ```xml
> <bean id="userService" class="org.leadfar.service.UserService"/>
> <bean id="documentService" class="org.leadfar.service.DocumentService"/>
> <bean id="orgService" class="org.leadfar.service.OrgService"/>
>  
> <bean id="userAction" class="org.leadfar.web.UserAction">
>      <property name="userService" ref="userService"/>
> </bean>
> ```
>
> 在Spring这个商店（工厂）中，有很多对象/服务：userService,documentService,orgService，也有很多会员：userAction等等，声明userAction需要userService即可，Spring将通过你给它提供的通道主动把userService送上门来，因此UserAction的代码示例类似如下所示：
>
>
> ```java
> package org.leadfar.web;
> public class UserAction{
>      private UserService userService;
>      public String login(){
>           userService.valifyUser(xxx);
>      }
>      public void setUserService(UserService userService){
>           this.userService = userService;
>      }
> }
> ```
>
> 在这段代码里面，你无需自己创建UserService对象（Spring作为背后无形的手，把UserService对象通过你定义的setUserService()方法把它主动送给了你，这就叫依赖注入！），当然咯，我们也可以使用注解来注入。`Spring依赖注入的实现技术是：动态代理`

**IoC解决了什么问题？**

思想就是两方之间不相互依赖，由第三方容易管理资源，好处：

对象之间的耦合度或者说依赖程度降低；

资源变得容易管理；比如你用Spring容器提供的话就很容易可以实现一个单例。



## AOP面向切面

**AOP：Aspect oriented programming 面向切面编程，AOP 是 OOP(面向对象编程)的一种延续。**

AOP是一种编程思想，在java中利用反射机制实现

事先只需要考虑主流程，不需要考虑哪些不重要的流程。

AOP不会把代码加到源文件中，但是它最终会正确的影响`机器代码`

**AOP 解决了什么问题**？

通过上面的分析可以发现，AOP 主要用来解决：在不改变原有业务逻辑的情况下，增强横切逻辑代码，根本上解耦合，避免横切逻辑代码重复。

**AOP 为什么叫面向切面编程**

切 ：指的是横切逻辑，原有业务逻辑代码不动，只能操作横切逻辑代码，所以面向横切逻辑

面 ：横切逻辑代码往往要影响的是很多个方法，每个方法如同一个点，多个点构成一个面。这里有一个面的概念

>  从Spring的角度看，AOP最大的用途就在于提供了事务管理的能力。事务管理就是一个关注点，你的正事就是去访问数据库，而你不想管事务（太烦），所以，Spring在你访问数据库之前，自动帮你开启事务，当你访问数据库结束之后，自动帮你提交/回滚事务！