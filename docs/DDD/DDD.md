## DDD

### 六边形架构
端口（Ports）：**类似service接口** 在示例中，ProductServicePort 接口代表了六边形模型中的端口，它定义了应用程序的核心逻辑可以对外提供哪些服务或操作。这些端口是领域逻辑与外部世界进行交互的入口和出口。

适配器（Adapters）：**controller和数据库的responsory，mq适配等** 在示例中，ProductController 类充当了一个适配器的角色，它将外部的 HTTP 请求转换为内部领域逻辑可以理解的操作，并将操作结果转换回 HTTP 响应。同样，Spring Data JPA 仓库实现也是一个适配器，它将领域逻辑与数据库之间的交互进行了适配。

领域逻辑（Domain Logic）：**Domani** 在示例中，ProductService 类实现了 ProductServicePort 接口，包含了商品的业务逻辑。这部分逻辑是应用程序的核心，它被端口和适配器所包围，与外部系统（如数据库、Web API等）解耦。

配置：六边形模型中的配置通常位于应用程序的外围，用于配置适配器如何与外部世界交互。在示例中，application.properties 文件包含了数据库连接和 Hibernate 配置，这些配置决定了如何与数据库适配器交互。

包结构：

```java
com.example.productservice
├── ProductServiceApplication.java (启动类)
├── domain (领域层，包含业务逻辑)
│   ├── Product.java (领域实体)
│   └── ProductRepository.java (领域仓库接口)
├── application (应用层，连接领域层和适配器)
│   ├── port (端口，定义内外交互的接口)
│   │   ├── in (输入端口，由应用核心实现)
│   │   │   └── ProductServicePort.java
│   │   └── out (输出端口，由外部适配器实现)
│   │       └── ProductPersistencePort.java
│   └── service (应用服务，实现输入端口接口)
│       └── ProductService.java
├── adapter (适配器层，连接应用层和外部资源)
│   ├── in (输入适配器，处理外部请求到应用层的映射)
│   │   └── web (Web适配器，例如REST控制器)
│   │       └── ProductController.java
│   └── out (输出适配器，处理应用层到外部资源的映射)
│       ├── persistence (持久化适配器，例如JPA实现)
│       │   └── ProductJpaRepository.java
│       └── messaging (消息适配器，例如Kafka生产者)
│           └── ProductEventPublisher.java
└── infrastructure (基础设施层，包含配置和跨切面关注点)
    ├── config (配置类，例如数据库配置)
    │   ├── DatabaseConfig.java
    │   └── MessagingConfig.java
    └── aspect (切面，例如日志或安全性)
        └── LoggingAspect.java
```

### 腐化层的作用
**是什么 ：应用不要直接依赖外域的信息，要把外域的信息转换成自己领域上下文（Context）的实体再去使用，从而实现本域和外部依赖的解耦**
举个例子，假如有一个电商系统，对于下单这个操作，它需要联动订单服务、商品服务、库存服务、营销服务等多个系统才能完成。
那么在订单域，该如何获取商品和库存信息呢？最直接的方式，无外乎就是RPC调用商品和库存服务，拿到DTO直接使用就完了。
然而，商品域吐出的是一个大而全的DTO（可能包含几十个字段），而在下单这个阶段，订单所需要的可能只是其中几个字段而已。更合适的做法，应该是在订单域中，使用gateway对商品域和库存域的依赖进行解耦。

1. 使用 CQRS：通过命令查询责任分离（CQRS）模式，将命令（写）和查询（读）逻辑分离。这样，复杂的查询逻辑可以放在专门的查询模型中，而不会影响领域模型。
2. 创建专门的查询服务：如上例中的 ProductQueryService，它专门处理查询，并将结果转换为 DTO。这个服务相当于一个腐化层，它封装了所有查询相关的逻辑，防止这些逻辑影响到领域模型。
3. 使用 DTO 来传输数据：通过 DTO 来传输数据，可以确保领域模型不会因为展示层或数据传输的需求而被迫改变。


阿里COLA，DDD的工程架构：https://developer.aliyun.com/article/861220
