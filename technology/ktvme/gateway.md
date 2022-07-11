# 中台网关技术选型方案

## 需求

需求原稿见附件。整理总结后可分为功能和性能需求两块。

功能需求为：

- 负载均衡
- 请求聚合
- 鉴权
- 请求过滤
- 限流（可视为请求过滤的一种场景）
- 动态路由
- 灰度发布
- Java / PHP 支持

性能要求为:

- 高可用
- 低延迟
- 便于维护性

上述功能几乎涵盖现存所有的网关解决方案的功能合集，且大部分解决方案都只能覆盖其中的一种或者数种功能。如果在开始阶段即要实现上述全部功能，可能需要多种方案的组合才可实现。无论从工期，技术风险，架构复杂度考虑，都不建议。因此我们首先咨询需求方，当前一期阶段最为关注且必要的功能有哪些，为了解决哪些具体问题因此需要引入网关中间件来进行支持。

除需求方以外，吴福财作为公司运维负责人，也从运维角度谈了运维人员对于网关的理解和期许。

运维角度：

- 限流：公司当前的限流功能基于 Nginx，只能对于二级域名生效，Nginx 通过日志监控，业务方实现具体熔断策略。如有可能最好能提供统一的规范和封装，便于在全平台支持以及未来的扩展。
- 网关：期望采用统一的技术架构，方便后续的开发与维护。
- 服务发现：当现有业务进行扩容时，运维通过工具和脚本实现从阿里云动态分配主机，获取 IP。但对应服务的 Nginx 配置，尤其是 load balance 相关的 upstrem 配置，仍然需要运维人员手工维护并修改，运行 Nginx Configration reload。以此来实现，节点扩展后，请求的流入与分发通过新节点运行。

业务方一期网关必要功能：

- 鉴权： 中台通过网关能实现统一鉴权。
- 路由： 现有中台服务还是由各种微加服务组成，但希望能通过网关暴露统一接口，供对外使用。内部微服务间的相互调用，仍可使用之前机制，但最好能支持服务发现与服务注册。
- 限流： 可视为鉴权/路由的一种极端形式，可根据需要实现接口级别的开启与关闭。

## 方案

将网关从功能上分为三个功能维度分别论述：

- 服务治理
- 网关路由与接口限流

### 1. 服务治理（服务发现/服务注册/负载均衡）

不久前测试环境切换，大家为了维护各个服务地址变更，集体部署，更新持续了数天之久。现在各个业务普遍使用参数表来维护依赖服务地址，这种导致批量变更时，无可避免的出现混乱与无序。大家都有共识，需要提供一种简单方便的机制对现有服务进行管理，避免后续出现上述场景。

运维也希望服务扩容时，能自动将服务的动态部署信息（IP，主机，端口）进行更新，并自动更新 Nginx 配置，自动更新 Nginx 运行信息。

#### Step1: 基于 Nacos 的静态地址管理

使用 Nacos 来管理各应用服务地址，由运维负责维护一份标准的[服务节点 => URL 域名]映射关系表。该表分为开发，测试，生产三个环境的不同映射数据。表数据，使用 Nacos 在各应用间进行分发和同步，替代当前各应用各自的参数表。各应用（Java、PHP）简单修改现有代码，将之前的参数表或者静态配置映射，进行简单修改，改为使用 Nacos 读入的环境参数即可。各应用间的相互调用仍然通过参数表配置的 URL 进行，变更的仅为 URL 获取方式。请求仍然通过 Nginx，也不会影响现有基于 Nginx 的日志统计，负载均衡等相关功能。

这种方式的优点是原始且稳定，可视为配置文件分发的一种新方式。此外，Nacos 作为阿里云的服务管理套件之一，也支持 K8s 容器管理。即使未来将服务升级至 K8S 容器化管理。阿里云仍然支持，且对于应用而言，也仅仅视为将 Nginx 地址变更为阿里云服务的 LBS 地址而已，整体机制不变。缺点为服务的添加，修改仍为手工运行（现阶段）。但正常新增服务一个月平均也不超过 1 个，足够满足当前规模内的管理需要。

#### Step 2: 基于 Nacos 的动态节点管理

服务发现

- Java：通过 SpringCloud 实现服务管理，将 Nacos 的动态节点管理接入 SpringCloud，实现服务发现与管理。
- PHP: 通过编写专用 Agent 监视 PHP 的服务节点部署或重启，通过 Nacos Http API 实现 PHP 服务的更新，监听。

节点调用：

- Java: 可以继续使用之前的服务静态表，也可以通过 SpringCloud RestTemple 来实现服务节点间互调，并实现 Client Load Balance.
- PHP：可以通过  Nacos Http API 实现服务节点调用。[参考文档](https://docs.ktvme.com/project-17/doc-55/) 但这种方式意味着 PHP 需要监听节点运行情况，对于 PHP 的 Curl 需要进行较大修改且有一定的性能损耗。

> 相比传统的 Nginx，使用 SpringCloud 方式进行服务调用收益有限，Client Load Balance 在实际使用中也有质疑。Eureka 的负载调节与响应速度相比 Nginx 仍相差甚远。各家的 LBS 解决方案都鲜少使用此种方式。最后，当服务升级至 K8S 容器化管理时，此种方式也会受到影响。因此我们和运维均认为此种方式并不推荐。此外，使用此项方案也意味着基于 Nginx 日志的调用统计也无法使用。

> 另外一个潜在问题是，现有 Java 程序中业务接口调用，代码必要要求使用 RestTemple 来实现。很多程序员更喜欢使用轻量级的 `OkHttpClient` 和 `Retrofit` 来实现 HttpCall。为使用 Spring Cloud 的 Client Load Balance, 上述代码都需要使用 `RestTemplate` 或 `WebClient` 来进行改写，因此使用此项机制也会要求代码进行相应修改。

综上所述，我们更推荐使用参数表和域名来实现服务调用，实际生产环境中，大数据环境已使用 Eureka 实现服务注册，但服务调用仍然通过域名来进行。但如果引入 Nacos 作为服务注册方案，对于小规模的探索和使用 Spring Cloud 进行服务访问方案，我们也持鼓励和推荐态度。

> NOTE: 当前测试环境和生产环境仍然使用的是较老的 Nacos 1.1.4 版本。该版本已知有一堆诸如安全，集群，SpringCLoud 版本支持问题。并不建议大家使用现有版本。后续运维会将现有服务升级至最新的 Nacos 2 版本，建议等待运维升级后再行接入。（由于安全设计问题，Nacos1， Nacos2 版本并不兼容）

#### Step 3: 基于阿里云 K8s 的容器管理方案

这将是服务管理的最终解决方案。所有的现有服务和未来服务都会被容器化，进行管理。Nginx 会被阿里云的 LBS 服务替换。容器的创建也注册均通过阿里云内部 API 来实现。包括 CI/CD 在内，阿里云提供全套工作栈支持。换运维估算，最快两个季度后就会开始前期业务接入。

如此项进展顺利，前述 step-2 就意义有限。

### 2. 网关路由和接口限流

网关路由当前主要解决中台各微服务 URL 一致性问题和请求认证授权问题。这两者当前都属于中台的业务范畴，因此，截止目前，网关路由部分建议先由中台负责单独建设。

当前，业界网关程序通常以Nginx 扩展，Golang，Spring Cloud 三种不同平台解决方案。当前，较少 PHP 系统化网关解决方案。大都基于 Lumen 和 Laravel 构建业务网关。

基于 Spring Cloud 的网关解决方案有二:

- Netflix Zuul
- Spring Cloud Gateway

后者作为前者的替代和更新，基于 Spring Framework 5、Spring Boot 2.0 版本。支持以下 feature：

- Path Rewriting
- Easy to write Predicates and Filters
- Predicates and filters are specific to routes
- Able to match routes on any request attribute

可很方便的以 FP 模式实现路由规则的编写。

此外，对于`接口限流`和`熔断`，Spring Cloud Gateway 也提供了相应支持:

- Request Rate Limiting
- Circuit Breaker integration.

综上所述，网关技术栈上推荐使用 Spring Cloud Gateway 进行实现。

优点：

- Spring Cloud 支持，即使后续不使用诸如 LBS 等机制，也方便集成现有 Java 诸微服务。
- Netflix Zuul、 Spring Cloud Gateway 都有完整社区支持与更新，也在诸多生产环境上使用
- 对于常用网关功能进行完整封装，无需从头开始实现

缺点：

- PHP 无法像 Spring Java 应用那样使用 Spring Cloud 进行方便的接入与集成。 但如前所述，我们推荐 PHP 仍使用传统的参数表方式来进行服务调用与接入，那么 PHP 也就不受影响：只要能接受从网关层路由过来的业务请求即可。
- 当处理 SSL 与 HTTP 同时支持，Spring Cloud Gateway 需要启用 InsecureTrustManager 以支持 SSL 证书
- Spring Cloud Gateway 基于 webflux 来实现，并非所有人能接受和喜爱 ReactiveX 风格技术栈。从这个方向来说，Netflix Zuul 使用传统的 Spring MVC Web Filters 封装，可能更为传统程序员所接受。

## 总结

综上所述，对于现阶段中台网关需求，我们建议使用 Spring Cloud Gateway 进行实现。此外，对于当前服务治理与服务发现，当前，推荐使用基于 Nacos 2 版本进行实现。后续还会补充基于 Spring Gateway 的 demo 代码，并基于 demo 的 ab 性能压测报告。