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
