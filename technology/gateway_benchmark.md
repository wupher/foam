# Gatedway deom banchemark

商户通的最新需求中要对单一用户多商家营业额展示提供连锁展示面板。由于同时需要计算多家商家的营业额统计，因此需要进行压测。测试环境通过从生产环境导入 100 个真实商家数据进行压测，按 1 个区域 10 商家进行区域分组设置。

Gateway 项目通过代理商户通的对应请求，然后通过 [K6](https://k6.io) 进行请求压测，观察 Gateway 性能变化。测试真实应用场景中，慢速请求和多用户并发对于代理程序的影响。

K6 脚本及运行结果已直接导入至 demo 项目中。拉取代码后即可查看，执行。

其中对 100 商家历史数据请求统计结果如下：

![系统截图](https://gitee.com/fanwu/spring-gateway-demo/raw/main/screen_shots/CleanShot%202022-05-11%20at%2017.30.37@2x.png)

此接口用于计算 100 商家，10 区域，时间跨度 1 年的数据报表展示。请求请求时间在 2.33 秒。属于常见的慢速请求之一。

对于 Gatway 程序的 CPU 影响，近乎于无。

![Gatedway HTOP截图](https://gitee.com/fanwu/spring-gateway-demo/raw/main/screen_shots/CleanShot%202022-05-11%20at%2014.24.42@2x.png)

大多数请求下代理程序并不受压力影响（它并不真正处理业务，只是转发）。由于使用 Netty 的异步非阻塞模式，因此可以观察到，大部分时候，请求上升时，gateway 的线程并发是同部署机器的 CPU 核数相同。更多的线程并不会让程序运行更快，这也是与传统的 Tomcat / Apache 并发模式的不同。

![100 商家实时模式请求](https://gitee.com/fanwu/spring-gateway-demo/raw/main/screen_shots/CleanShot%202022-05-11%20at%2017.32.14@2x.png)

这是另外一种实时模式业务统计场景，Gatedway 情况类似，这里不再赘述。

建议使用 Gateway 设置自己需要关心的业务场景，进行相应的压力测试并进行评估。
