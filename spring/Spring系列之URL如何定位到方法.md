# Spring系列之URL如何定位到方法

分两步：

1、Spring容器启动的时候。

扫描 加了  `@Controller` 注解的类。

然后查找加了 `@RequestMapping` 注解的方法。

并将这些方法，进行注册，就是加入到 `map` 中。

涉及到的类、接口或者方法：

HandlerMapping

AbstractHandlerMethodMapping implements InitializingBean

afterPropertiesSet()

initHandlerMethods();



2、当发起请求时，需要处理这些请求

getHandlerInternal(HttpServletRequest request)

