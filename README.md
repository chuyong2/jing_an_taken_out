![项目使用技术](https://user-images.githubusercontent.com/88364565/197378938-d3d29ad6-245d-4924-8968-618bb38cc7b4.png)
### 学习项目收获：
 - 了解企业项目开发的完整流程，增长开发经验
 - 了解需求分析的过程，提高分析和设计能力
 - 对所学技术进行灵活应用，提高编码能力
 - 解决各种异常情况，提高代码调试能力
## 项目介绍
本项目是专门为餐饮企业（餐厅、饭店）定制的一款软件产品，包括系统管理后台和移动端应用两部分。其中系统管理后台主要提供给餐饮企业内部员工使用，可以对餐厅的菜品、套餐、订单等进行管理维护。移动端应用主要提供给消费者使用，可以在线浏览菜品、添加购物车、下单等。
## 功能结构
### 移动端前台
 - 手机号登录     
 - 微信登陆      
 - 地址管理       
 - 历史订单       
 - 菜品规格
 - 菜品规格
 - 购物车
 - 下单
 - 菜品浏览
### 系统管理后台
 - 分类管理
 - 菜品管理
 - 套餐管理
 - 菜品口味管理
 - 员工登录
 - 员工退出
 - 员工管理
 - 订单管理
## 角色
 - 后台系统管理员：登录后台管理系统，拥有后台系统中的所有操作权限
 - 后台系统普通员工：登录后台管理系统，对菜品、套餐、订单等进行管理
 - C端用户：登录移动端应用，可以浏览菜品、添加购物车、设置地址、在线下单等
## 数据库设计
![image](https://user-images.githubusercontent.com/88364565/197379518-86b5d20f-5b27-4528-87b8-9e9d0ab70f6f.png)
## 后台登录功能开发
### 需求分析
查看登录请求信息，通过浏览器调试工具（F12），可以发现，点击登录按钮时，页面会发送请求（请求地址为http://localhost:/8080/employee/login）并提交参数（username和password）
此时报404，是因为我们的后台系统还没有响应此请求的处理器，所以我们需要创建相关类来处理登录请求
![image](https://user-images.githubusercontent.com/88364565/197379726-14746db8-3805-4ca8-b1c5-6d7194c94b38.png)
### 代码开发
处理逻辑如下：
 - 将页面提交的密码password进行md5加密处理                                   ![image](https://user-images.githubusercontent.com/88364565/197379942-cf6422b6-172a-4d43-9760-4551f275e6ab.png)
 - 根据页面提交的用户名username查询数据库
 - 如果没有查询到则返回登录失败结果
 - 密码比对，如果不一致则返回登录失败结果
 - 查看员工状态，如果为已禁用状态，则返回员工已禁用结果
 - 登录成功，将员工id存入Session并返回登录成功结果
## 后台退出功能开发
### 需求分析
员工登录成功后，页面跳转到后台系统首页面（backend/index.html），此时会显示当前登录用户的姓名，如果员工需要退出系统，直接点击石侧的退出按钮即可退出系统，退出系统后页面应跳转回登录页面
### 代码开发
 用户点击页面中退出按钮，发送求，请求地址为/employee/logout，请求方式为POST。我们只需要在Controller中创建对应的处理方法即可，具体的处理逻辑
 - 清理Session中的用户id
 - 返回结果
## 完善登录功能（拦截）
### 问题分析                                              Q
前面我们已经完成了后台系统的员工登录功能开发，但是还存在一个问题：用户如果不登录，直接访问系统首页面，照样可以正常访问。
这种设计并不合理，我们希望看到的效果应该是，只有登录成功后才可以访问系统中的页面，如没有登录则跳转到登录页面。
那么，具体应该怎么实现呢
答案就是使用过滤器或者拦器，在过滤器或者拦载器中判断用户是否已经完成登录， 如没有登录则跳转到登录页面
### 代码实现
 - 创建自定义过滤器LogincheckFilter
 - 在启动类上加入注解@servletcomponentscan
 - 完善过滤器的处理逻辑
![image](https://user-images.githubusercontent.com/88364565/197380404-8d977ad1-ab9e-44b0-b100-f3d689c6bd90.png)

过滤器具体的处理逻辑如下：
 - 获取本次请求的URI
 - 判断本次请求是否需要处理
 - 如果不需要处理，则直接放行
 - 判断登录状态，如果已登录，则直接放行
 - 如果未登录则返回未登录结果
# 员工管理业务开发
## 新增员工
### 数据模型
新增员工，其实就是将我们新增页面录入的员工数据插入到employee表。需要注意，employee表中对username字段加入了唯一约束，因为username是员工的登录账号，必须是唯一的
status默认值为1（1为正常，0为未启用）
### 代码开发
在开发代码之前，需要梳理一下整个程序的执行过程
 - 页面发送ajax请求，将新增员工页面中输入的数据以json的形式提交到服务端
 - 服务端Controller接收页面提交的数据并调用Service将数据进行保存
 - Service调用Mapper操作数据库，保存数据
![image](https://user-images.githubusercontent.com/88364565/197380641-dcb701e6-35c1-49ea-8e01-c772763fdf21.png)
### 发现问题
前面的程序还存在一个问题，就是当我们在新增员工时输入的账号已经存在，由于employee表中对该字段加入了唯一约束，此时程序会抛出异常：java. sql. SQLIntegrityConstraintViolationException: Duplicate entryzhangsan for keyidx_username。此时需要我们的程序进行异常捕获，通常有两种处理方式
 - 在Controller方法中加入try、catch进行异常捕获 
 - 使用异常处理器进行全局异常获
### 小结
1、根据产品原型明确业务需求
2、重点分析数据的流转过程和数据格式
3.通过debug断点调试跟踪程序执行过程
## 员工信息分页查询
员工数据多了就要分页显示，否则杂乱无章
### 代码开发
在开发代码之前，需要梳理一下整个程序的执行过程
 - 页面发送ajax请求，将分页查询参数（page、pagesize、name）提交到服务端
 - 服务端Controller接收页面提交的数据并调用ervice查询数据
 - Service调用Mapper操作数据库，查询分页数据
 - Controller将查询到的分页数据响应给页面
 - 页面接收到分页数据并通过Elementul的Table组件展示到页面上
![image](https://user-images.githubusercontent.com/88364565/197380958-90741c70-e720-4096-979e-4955fd2f2184.png)
## 启用禁用员工账号
### 需求分析
在员工管理列表页面，可以对某个员工账号进行启用或者禁用操作。账号禁用的员工不能登录系统，启用后的员工可以正常登录
需要注意，只有管理员（admin用户）可以对其他普通用户进行启用、禁用操作，所以普通用户登录系统后启用禁用按钮不显示。
管理员admin登录系统可以对所有员工账号进行启用、禁用操作
如果某个员工账号状态为正常，则按钮显示为“禁用”，如果员工账号状态为已禁用，则按钮显示为“启用”，普通用户登录则不显示按钮
![image](https://user-images.githubusercontent.com/88364565/197381188-a695f3a2-8c35-43f8-b17c-c115e77b4a08.png)
### 代码开发
在开发代码之前，需要梳理一下整个程序的执行过程
 - 页面发送ajax请求，将参数（id、status）提交到服务端
 - 服务端Controller接收页面提交的数据并调用Service更新数据
 - Service调用Mapper操作数据库
![image](https://user-images.githubusercontent.com/88364565/197381232-aa770ac4-e3ce-4d93-87cd-bde14804b160.png)
### 功能测试
![image](https://user-images.githubusercontent.com/88364565/197381272-9968d390-ad02-42e8-8681-fbe2f81890ec.png)
### 代码修复
前面我们已经发现了问题的原因，即js对long型数据进行处理时丢失精度，导致提交的id和数据库中的id不一致。如何解决这个问题？
我们可以在服务端给页面响应json数据时进行处理，将long型数据统一转为String字符串。
### 实现
 - 提供对象转换器jacksonObjectMapper，基于jackson进行java对象到json数据的转换（资料中已经提供，直接复制到项目中使用）
 - 在WebMvcConfig配置类中扩展Springmvc的消息转换器，在此消息转换器中使用提供的对象转换器进行ava对象到json数据的转换
## 编辑员工信息
### 代码开发
在开发代码之前需要梳理一下操作过程和对应的程序的执行流程
 - 点击编辑按钮时，页面跳转到add.html，并在url中携带参数员工id
 - 在add.htm页面获取url中的参数员工id]
 - 发送ajax请求，请求服务端，同时提交员工id参数
 - 服务端接收请求，根据员工id查询员工信息，将员工信息以json形式响应给页面
 - 页面接收服务端响应的json数据，通过VUE的数据绑定进行员工信息回显
 - 点击保存按钮，发送ajax请求，将页面中的员工信息以json方式提交给服务端
 - 服务端接收员工信息，并进行处理，完成后给页面响应
 - 页面接收到服务端响应信息后进行相应处理
注意：add.html为公共页面，新增和编辑都在其中实现
# 分类管理业务开发
## 公共字段自动填充
### 问题分析
前面我们已经完成了后台系统的员工管理功能开发，在新增员工时需要设置创建时间、创建人、修改时间、修改人等字段，在编辑员工时需要设置修改时间和修改人等字段。这些字段属于公共字段，也就是很多表中都有这些字段，如下：
![image](https://user-images.githubusercontent.com/88364565/197381778-039fe536-0571-45ee-8de4-e2e5dd457d02.png)
使用MybatisPlus可以简化开发，统一处理
### 代码实现
MybatisPlus公共字段自动填充，也就是在插入或者更新的时候为指定字段赋予指定的值，使用它的好处就是可以统一对这些字段进行处理，避免了重复代码，实现步骤
 - 在实体类的属性上加入@TableField注解，指定自动填充的策略
 - 按照框架要求编写元数据对象处理器，在此类中统一为公共字段赋值，此类需要实现MetaobjectHandler接口
![image](https://user-images.githubusercontent.com/88364565/197381871-a7bbdc59-c6b5-43a0-8d22-c6a0707cc92a.png)
![image](https://user-images.githubusercontent.com/88364565/197381890-8f727de7-f29e-4df0-bf59-343a506299d4.png)
### 功能完善
前面我们已经完成了公共字段自动填充功能的代码开发，但是还有一个问题没有解决，就是我们在自动填充createUser和updateUser时设置的用户id是固定值，现在我们需要改造成动态获取当前登录用户的id。
有的同学可能想到，用户登录成功后我们将用户id存入了Httpsession中，现在我从Httpsession中获取不就行了？
注意，我们在MyMetaObiectHandler类中是不能获得Httpsession对象的，所以我们需要通过其他方式来获取登录用户id可以使用ThreadLocal来解决此问题，它是JDK中提供的一个类
在学习ThreadLocal之前，我们需要先确认一个事情，就是客户端发送的每次http请求，对应的在服务端都会分配一个新的线程来处理， 在处理过程中涉及到下面类中的方法都属于相同的一个线程；
 - LogincheckFilter的doFilter方法
 - EmployeeController的update方法
 - MyMetaobjectHandler的updateFil方法
 可以在上面的三个方法中分别加入下面代码（获取当前线程id）
 ```
  long id = Thread.currentThread().getId();
  log.info("线程id:{}"，id);
 ```
  执行编辑员工功能进行验证，通过观察控制台输出可以发现，一次请求对应的线程id是相同的
![image](https://user-images.githubusercontent.com/88364565/197382033-2269f513-fe29-4dbd-8d00-0569834e09fd.png)
### 什么是ThreadLocal:
ThreadLocal并不是一个Thread，而是Thread的局部变量。当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。ThreadLocal为每个线程提供单独一份存储空间，具有线程隔离的效果，只有在线程内才能获取到对应的值，线程外则不能访问。

ThreadLocal常用方法：
   public void set(T value）   设置当前线程的线程局部变量的值
   public T get()               返回当前线程所对应的线程局部变量的值
   
我们可以在LogincheckFilter的doFilter方法中获取当前登录用户id，并调用ThreadLocal的set方法来设置当前线程的线
程局部变量的值（用户id），然后在MyMetaObjectHandler的updateFil方法中调用ThreadLocal的get方法来获得当前
线程所对应的线程局部变量的值（用户id）
### 实现步骤
 - 编写Basecontext工具类，基于ThreadLocal封装的工具类
 - 在LogincheckFilter的doFilter方法中调用BaseContext来设置当前登录用户的id
 - 在MyMetaObjectHandler的方法中调用BaseContext获取登录用户的id
## 新增分类
### 需求分析
后台系统中可以管理分类信息，分类包括两种类型，分别是菜品分类和套餐分类。当我们在后台系统中添加菜品时需要选择一个菜品分类，当我们在后台系统中添加一个套餐时需要选择一个套餐分类，在移动端也会按照菜品分类和套餐分类来展示对应的菜品和套餐。
### 数据模型
新增分类即：在新增窗口录入的分类数据插入到category中，表中name唯一。
### 代码开发
在开发业务功能前，先将需要用到的类和接口基本结构创建好
 - 实体类Category
 - Mapper接口CategoryMapper
 - 业务层接口Categoryservice
 - 业务层实现类Categoryservicelmpl
 - 控制层Categorycontroller
在开发代码之前，需要梳理一下整个程序的执行过程
 - 页面（backend/page/category/list.html)发送ajax请求，将新增分类窗口输入的数据以json形式提交到服务端
 - 服务端Controller接收页面提交的数据并调用Service将数据进行保存
 - Service调用Mapper操作数据库，保存数据
可以看到新增菜品分类和新增套餐分类请求的服务端地址和提交的json数据结构相同，所以服务端只需要提供一个方法统一处理即可
![image](https://user-images.githubusercontent.com/88364565/197382955-c3f68995-20d4-479a-8190-bd13d47bddb8.png)
## 分类分页查询
### 代码开发
在开发代码之前，需要梳理一下整个程序的执行过程
 - 页面发送ajax请求，将分页查询参数（page、pagesize）提交到服务端
 - 服务端Controller接收页面提交的数据并调用Service查询数据
 - Service调用Mapper操作数据库，查询分页数据
 - Controller将查询到的分页数据响应给页面
 - 页面接收到分页数据并通过Element-ul的Table组件展示到页面上
![image](https://user-images.githubusercontent.com/88364565/197383557-e405f31f-7fb4-40b9-b444-8f8de2d28754.png)
## 删除分类
### 分析
在分类管理列表页面，可以对某个分类进行删除操作。需要注意的是当分类关联了菜品或者套餐时，此分类不允许删除
### 代码开发
 在开发代码之前，需要梳理一下整个程序的执行过程
 - 页面发送aiax请求，将参数（id）提交到服务端
 - 服务端Controller接收页面提交的数据并调用Service删除数据
 - Service调用Mapper操作数据库
![image](https://user-images.githubusercontent.com/88364565/197384133-31485593-3453-41fa-b3c4-7e7582fd0e6e.png)
### 功能完善
前面我们已经实现了根据id删除分类的功能，但是并没有检查删除的分类是否关联了菜品或者套餐，所以我们需要进行功能完善。
要完善分类删除功能，需要先准备基础的类和接口
 - 实体类Dish和setmeal
 - Mapper接口DishMapper和SetmealMapper
 - Service接口Dishservice和Setmealservice
 - Service实现类Dishservicelmpl和SetmealserviceImpl
## 修改分类
### 分析
在分类管理列表页面点击修改按钮，弹出修改窗口，在修改窗口回显分类信息并进行修改，最后点击确定按钮完成操作。
# 菜品管理业务开发
## 文件上传下载






