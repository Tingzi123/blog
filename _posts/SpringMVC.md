title: SpringMVC
---
1、springMVC
spring结构：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springmvc1.png?raw=true)

SpringMVC是Spring的一个模块，是基于MVC的web框架
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springmvc2.png?raw=true)
1）用户端向前端控制器（DispatcherServlet)发起request请求
2）前端控制器向处理器映射器（HandlerMapping）发出请求查找处理器（Handler）（可根据XML和注解进行查找）
3）处理器映射器返回一个执行链给前端控制器
4）前端控制器请求处理器适配器（HandlerAdapter）执行handler
5）处理器适配器执行handler处理器
6）handler处理器返回ModelAndView给处理器适配器（ModelAndView是springmvc框架的一个底层对象）
7）处理器适配器将ModelAndView返回给前端控制器
8）前端控制器请求视图解析器（ViewResolver）进行视图解析（根据逻辑视图名解析成真正的视图）
9）视图解析器将view返回给前端控制器
10）前端控制器请求视图（view）进行视图渲染（视图渲染将模型数据（在ModelAndView对象中）填充到request域）
11）前端控制器向用户响应结果

重要组件：
1）前端控制器（DispatcherServlet)：接受请求，响应结果，相当于转发器,通过前端控制器来交互，减少其他组件之间的耦合度
2）处理器映射器（HandlerMapping）：根据请求的url查找handler
3）处理器适配器（HandlerAdapter）：按照特定的规则去执行handler
4）处理器（handler）
注意：编写handler时按照handlerAdapter的要求去做才能正确执行handler
5）视图解析器（ViewResolver）：根据逻辑视图名解析成真正的视图（view）
6）视图（View）：是一个接口，通过实现类来支持不同的view类型（jsp、pdf......)

例子：
