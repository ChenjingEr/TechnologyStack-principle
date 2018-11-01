Spring MVC的总体执行流程:
![ExecutionChain](img/执行流程.png)

到Spring MVC的入口由DispatcherServlet开始。

DispatcherServelt的类继承结构

![dispatcherServlet](img/dispatcherServlet_hierarchy.png)

DispatcherServelt是一个Servlet,初始化由init开始。DispatcherServlet链上的init在HttpServletBean中。
