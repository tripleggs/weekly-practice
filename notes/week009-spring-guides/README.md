# WEEK009 - Spring 官方入门指南

[Spring 官方提供了大量的教程文档](https://spring.io/guides)，方便初学者快速入门，并且每篇教程都极具操作性，我们可以一步一步按顺序进行体验。这些教程被官方分为了三类：

* Getting Started Guides（耗时 15 到 30 分钟，完成一个 Hello World 类型的项目）
* Topical Guides（需要至少 1 小时的阅读和理解，提供一些覆盖面更广的内容）
* Tutorials（需要 2 - 3 小时才能完成，提供一些更深入的主题，并且更适用于真实场景）

不过我感觉这种分类方式并不是很友好，由于官方文档特别多，所以看上去显得乱糟糟的。我希望通过教程的内容来进行分类，在这篇笔记中，我将尝试着把 Spring 官方教程全部体验一遍，并按照我的理解来重新分类。

## 开始

这部分内容介绍了如何使用常用的 IDE 工具创建一个 Spring Boot 项目脚手架，包括：`Spring Tool Suite`、`IntelliJ IDEA` 和 `Visual Studio Code`。

* [Working a Getting Started guide with STS](https://spring.io/guides/gs/sts/) - [*学习笔记*](./guides/gs/sts/README.md)
* [Working a Getting Started guide with IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/) - [*学习笔记*](./guides/gs/intellij-idea/README.md)
* [Building a Guide with VS Code](https://spring.io/guides/gs/guides-with-vscode/) - [*学习笔记*](./guides/gs/vscode/README.md)

## 项目构建

这部分内容介绍了如何使用 Maven 或 Gradle 来构建 Spring Boot 项目。

* [Building Java Projects with Maven](https://spring.io/guides/gs/maven/) - [*学习笔记*](./guides/gs/maven/README.md)
* [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle/) - [*学习笔记*](./guides/gs/gradle/README.md)

## Spring MVC

* [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)

## Spring Boot

* [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
* [Creating a Multi Module Project](https://spring.io/guides/gs/multi-module/)
* [Converting a Spring Boot JAR Application to a WAR](https://spring.io/guides/gs/convert-jar-to-war/)
* [Creating Asynchronous Methods](https://spring.io/guides/gs/async-method/)

### Restful Web Service

这部分内容介绍了如何使用 Spring Boot 构建一个简单的 Restful 的 Web 服务，并使用 `RestTemplate` 或前端 JavaScript 来调用它。

* [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) - [*学习笔记*](./guides/gs/rest-service/README.md)
* [【Tutorials】Building REST services with Spring](https://spring.io/guides/tutorials/rest/)
* [Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest/) - [*学习笔记*](./guides/gs/consuming-rest/README.md)
* [Consuming a RESTful Web Service with AngularJS](https://spring.io/guides/gs/consuming-rest-angularjs/)
* [Consuming a RESTful Web Service with jQuery](https://spring.io/guides/gs/consuming-rest-jquery/)

### Hypermedia-Driven RESTful Web Service

* [Building a Hypermedia-Driven RESTful Web Service](https://spring.io/guides/gs/rest-hateoas/)

### Spring Boot Actuator

* [Building a RESTful Web Service with Spring Boot Actuator](https://spring.io/guides/gs/actuator-service/)

### Basics

* [Handling Form Submission](https://spring.io/guides/gs/handling-form-submission/)
* [Validating Form Input](https://spring.io/guides/gs/validating-form-input/)
* [Uploading Files](https://spring.io/guides/gs/uploading-files/)

### SOAP

* [Producing a SOAP web service](https://spring.io/guides/gs/producing-web-service/)
* [Consuming a SOAP web service](https://spring.io/guides/gs/consuming-web-service/)

### CORS

* [Enabling Cross Origin Requests for a RESTful Web Service](https://spring.io/guides/gs/rest-service-cors/)

### Reactive

* [Building a Reactive RESTful Web Service](https://spring.io/guides/gs/reactive-rest-service/)

### Kotlin

* [【Tutorials】Building web applications with Spring Boot and Kotlin](https://spring.io/guides/tutorials/spring-boot-kotlin/)
* [【Tutorials】Spring Boot with Kotlin Coroutines and RSocket](https://spring.io/guides/tutorials/spring-webflux-kotlin-rsocket/)

## WebSocket

* [Using WebSocket to build an interactive web application](https://spring.io/guides/gs/messaging-stomp-websocket/)

## 访问数据

* [Accessing Data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
* [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
* [Accessing Relational Data using JDBC with Spring](https://spring.io/guides/gs/relational-data-access/)
* [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
* [Accessing Data with Cassandra](https://spring.io/guides/gs/accessing-data-cassandra/)
* [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/)
* [Accessing Data in Pivotal GemFire](https://spring.io/guides/gs/accessing-data-gemfire/)
* [Accessing Data with R2DBC](https://spring.io/guides/gs/accessing-data-r2dbc/)
* [Accessing Data Reactively with Redis](https://spring.io/guides/gs/spring-data-reactive-redis/)
* [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest/)
* [Accessing Neo4j Data with REST](https://spring.io/guides/gs/accessing-neo4j-data-rest/)
* [Accessing MongoDB Data with REST](https://spring.io/guides/gs/accessing-mongodb-data-rest/)
* [Accessing Data in Pivotal GemFire with REST](https://spring.io/guides/gs/accessing-gemfire-data-rest/)
* [Accessing Vault](https://spring.io/guides/gs/accessing-vault/)
* [Vault Configuration](https://spring.io/guides/gs/vault-config/)
* [Creating CRUD UI with Vaadin](https://spring.io/guides/gs/crud-with-vaadin/)
* [Integrating Data](https://spring.io/guides/gs/integration/)
* [【Tutorials】React.js and Spring Data REST](https://spring.io/guides/tutorials/react-and-spring-data-rest/)

## 事务处理

* [Managing Transactions](https://spring.io/guides/gs/managing-transactions/)

## 缓存

* [Caching Data with Spring](https://spring.io/guides/gs/caching/)
* [Caching Data with Pivotal GemFire](https://spring.io/guides/gs/caching-gemfire/)

## 消息通信

* [Messaging with Redis](https://spring.io/guides/gs/messaging-redis/)
* [Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)
* [Messaging with JMS](https://spring.io/guides/gs/messaging-jms/)
* [Messaging with Google Cloud Pub/Sub](https://spring.io/guides/gs/messaging-gcp-pubsub/)

## 安全认证

* [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
* [Authenticating a User with LDAP](https://spring.io/guides/gs/authenticating-ldap/)
* [【Topical Guides】Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture/)
* [【Tutorials】Spring Security and Angular](https://spring.io/guides/tutorials/spring-security-and-angular-js/)
* [【Tutorials】Spring Boot and OAuth2](https://spring.io/guides/tutorials/spring-boot-oauth2/)

## Spring Cloud 微服务

* [Service Registration and Discovery](https://spring.io/guides/gs/service-registration-and-discovery/)
* [Centralized Configuration](https://spring.io/guides/gs/centralized-configuration/)
* [Building a Gateway](https://spring.io/guides/gs/gateway/)
* [Client-Side Load-Balancing with Spring Cloud LoadBalancer](https://spring.io/guides/gs/spring-cloud-loadbalancer/)
* [Spring Cloud Stream](https://spring.io/guides/gs/spring-cloud-stream/)
* [Spring Cloud Data Flow](https://spring.io/guides/gs/spring-cloud-dataflow/)
* [Spring Cloud Task](https://spring.io/guides/gs/spring-cloud-task/)
* [Spring Cloud Circuit Breaker Guide](https://spring.io/guides/gs/cloud-circuit-breaker/)
* [Consumer Driven Contracts](https://spring.io/guides/gs/contract-rest/)

## 日志与监控

* [Observability with Spring](https://spring.io/guides/gs/tanzu-observability/)
* [【Tutorials】Metrics and Tracing with Spring](https://spring.io/guides/tutorials/metrics-and-tracing/)

## 容器与服务上云

* [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/) - [*学习笔记*](./guides/gs/spring-boot-docker/README.md)
* [Spring Boot Kubernetes](https://spring.io/guides/gs/spring-boot-kubernetes/)
* [Deploying a Spring Boot app to Azure](https://spring.io/guides/gs/spring-boot-for-azure/)
* [Deploying to Cloud Foundry from STS](https://spring.io/guides/gs/sts-cloud-foundry-deployment/)
* [【Topical Guides】Spring Boot Docker](https://spring.io/guides/topicals/spring-boot-docker/) - [*学习笔记*](../week011-spring-boot-on-docker/README.md)
* [【Topical Guides】Spring on Kubernetes](https://spring.io/guides/topicals/spring-on-kubernetes/)

## API 文档

* [Creating API Documentation with Restdocs](https://spring.io/guides/gs/testing-restdocs/)

## 测试

* [Testing the Web Layer](https://spring.io/guides/gs/testing-web/)

## 定时任务

* [Scheduling Tasks](https://spring.io/guides/gs/scheduling-tasks/)

## 批处理

* [Creating a Batch Service](https://spring.io/guides/gs/batch-processing/)

## 参考

1. [Spring Guides](https://spring.io/guides)

## 更多

### 1. Spring 官方文档

* [Spring Boot](https://spring.io/projects/spring-boot)
* [Spring Framework](https://spring.io/projects/spring-framework)
* [Spring Data](https://spring.io/projects/spring-data)
    * [Spring Data JDBC](https://spring.io/projects/spring-data-jdbc)
    * [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
    * [Spring Data LDAP](https://spring.io/projects/spring-data-ldap)
    * [Spring Data MongoDB](https://spring.io/projects/spring-data-mongodb)
    * [Spring Data Redis](https://spring.io/projects/spring-data-redis)
    * [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc)
    * [Spring Data REST](https://spring.io/projects/spring-data-rest)
    * [Spring Data for Apache Cassandra](https://spring.io/projects/spring-data-cassandra)
    * [Spring Data for Apache Geode](https://spring.io/projects/spring-data-geode)
    * [Spring Data for Apache Solr](https://spring.io/projects/spring-data-solr)
    * [Spring Data for VMware Tanzu GemFire](https://spring.io/projects/spring-data-gemfire)
    * [Spring Data Couchbase](https://spring.io/projects/spring-data-couchbase)
    * [Spring Data Elasticsearch](https://spring.io/projects/spring-data-elasticsearch)
    * [Spring Data Envers](https://spring.io/projects/spring-data-envers)
    * [Spring Data Neo4j](https://spring.io/projects/spring-data-neo4j)
    * [Spring Data JDBC Extensions](https://spring.io/projects/spring-data-jdbc-ext)
    * [Spring for Apache Hadoop](https://spring.io/projects/spring-hadoop)
* [Spring Cloud](https://spring.io/projects/spring-cloud)
    * [Spring Cloud Azure](https://spring.io/projects/spring-cloud-azure)
    * [Spring Cloud Alibaba](https://spring.io/projects/spring-cloud-alibaba)
    * [Spring Cloud for Amazon Web Services](https://spring.io/projects/spring-cloud-aws)
    * [Spring Cloud Bus](https://spring.io/projects/spring-cloud-bus)
    * [Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker)
    * [Spring Cloud CLI](https://spring.io/projects/spring-cloud-cli)
    * [Spring Cloud for Cloud Foundry](https://spring.io/projects/spring-cloud-cloudfoundry)
    * [Spring Cloud - Cloud Foundry Service Broker](https://spring.io/projects/spring-cloud-cloudfoundry-service-broker)
    * [Spring Cloud Cluster](https://spring.io/projects/spring-cloud-cluster)
    * [Spring Cloud Commons](https://spring.io/projects/spring-cloud-commons)
    * [Spring Cloud Config](https://spring.io/projects/spring-cloud-config)
    * [Spring Cloud Connectors](https://spring.io/projects/spring-cloud-connectors)
    * [Spring Cloud Consul](https://spring.io/projects/spring-cloud-consul)
    * [Spring Cloud Contract](https://spring.io/projects/spring-cloud-contract)
    * [Spring Cloud Function](https://spring.io/projects/spring-cloud-function)
    * [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)
    * [Spring Cloud GCP](https://spring.io/projects/spring-cloud-gcp)
    * [Spring Cloud Kubernetes](https://spring.io/projects/spring-cloud-kubernetes)
    * [Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix)
    * [Spring Cloud Open Service Broker](https://spring.io/projects/spring-cloud-open-service-broker)
    * [Spring Cloud OpenFeign](https://spring.io/projects/spring-cloud-openfeign)
    * [Spring Cloud Pipelines](https://spring.io/projects/spring-cloud-pipelines)
    * [Spring Cloud Schema Registry](https://spring.io/projects/spring-cloud-schema-registry)
    * [Spring Cloud Security](https://spring.io/projects/spring-cloud-security)
    * [Spring Cloud Skipper](https://spring.io/projects/spring-cloud-skipper)
    * [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth)
    * [Spring Cloud Stream](https://spring.io/projects/spring-cloud-stream)
    * [Spring Cloud Stream Applications](https://spring.io/projects/spring-cloud-stream-applications)
    * [Spring Cloud Task](https://spring.io/projects/spring-cloud-task)
    * [Spring Cloud Task App Starters](https://spring.io/projects/spring-cloud-task-app-starters)
    * [Spring Cloud Vault](https://spring.io/projects/spring-cloud-vault)
    * [Spring Cloud Zookeeper](https://spring.io/projects/spring-cloud-zookeeper)
    * [Spring Cloud App Broker](https://spring.io/projects/spring-cloud-app-broker)
* [Spring Cloud Data Flow](https://spring.io/projects/spring-cloud-dataflow)
* [Spring Security](https://spring.io/projects/spring-security)
    * [Spring Security Kerberos](https://spring.io/projects/spring-security-kerberos)
    * [Spring Security OAuth](https://spring.io/projects/spring-security-oauth)
    * [Spring Security SAML](https://spring.io/projects/spring-security-saml)
* [Spring for GraphQL](https://spring.io/projects/spring-graphql)
* [Spring Session](https://spring.io/projects/spring-session)
    * [Spring Session Core](https://spring.io/projects/spring-session-core)
    * [Spring Session Data Redis](https://spring.io/projects/spring-session-data-redis)
    * [Spring Session JDBC](https://spring.io/projects/spring-session-jdbc)
    * [Spring Session Hazelcast](https://spring.io/projects/spring-session-hazelcast)
    * [Spring Session MongoDB](https://spring.io/projects/spring-session-data-mongodb)
    * [Spring Session for Apache Geode](https://spring.io/projects/spring-session-data-geode)
* [Spring Integration](https://spring.io/projects/spring-integration)
* [Spring HATEOAS](https://spring.io/projects/spring-hateoas)
* [Spring REST Docs](https://spring.io/projects/spring-restdocs)
* [Spring Batch](https://spring.io/projects/spring-batch)
* [Spring AMQP](https://spring.io/projects/spring-amqp)
* [Spring CredHub](https://spring.io/projects/spring-credhub)
* [Spring Flo](https://spring.io/projects/spring-flo)
* [Spring for Apache Kafka](https://spring.io/projects/spring-kafka)
* [Spring LDAP](https://spring.io/projects/spring-ldap)
* [Spring Shell](https://spring.io/projects/spring-shell)
* [Spring Statemachine](https://spring.io/projects/spring-statemachine)
* [Spring Vault](https://spring.io/projects/spring-vault)
* [Spring Web Flow](https://spring.io/projects/spring-webflow)
* [Spring Web Services](https://spring.io/projects/spring-ws)

### 2. Github 上关于 Java 和 Spring 的学习资源

#### Java

* 【2.3K】https://github.com/dunwu/javacore
* 【3.1K】https://github.com/singgel/JAVA
* 【122K】https://github.com/Snailclimb/JavaGuide
* 【1.1K】https://github.com/superhj1987/pragmatic-java-engineer
* 【63.5k】https://github.com/doocs/advanced-java
* 【1.3K】https://github.com/Jackson0714/PassJava-Platform
* 【0.8K】https://github.com/caison/java-knowledge-mind-map
* 【4.3K】https://github.com/javagrowing/JGrowing
* 【3.6K】https://github.com/RedSpider1/concurrent
* 【1.3K】https://github.com/Jstarfish/JavaKeeper
* 【3.7K】https://github.com/RongleXie/java-books-collections
* 【1.4K】https://github.com/DreamCats/java-notes
* 【1.1K】https://github.com/dunwu/java-tutorial
* 【1.1K】https://github.com/DuHouAn/Java
* 【1.8K】https://github.com/kanwangzjm/funiture
* 【1.1K】https://github.com/Wasabi1234/Java-Concurrency-Progamming-Tutorial
* 【2.4K】https://github.com/zq2599/blog_demos
* 【1.0K】https://github.com/lenve/javaboy-code-samples

#### Spring

* 【23.6K】https://github.com/wuyouzhuguli/SpringAll
* 【27.5K】https://github.com/ityouknow/spring-boot-examples
* 【25.7K】https://github.com/xkcoding/spring-boot-demo
* 【5.6K】https://github.com/527515025/springBoot
* 【1.7K】https://github.com/javahongxi/whatsmars
* 【1.1K】https://github.com/chengxy-nds/Springboot-Notebook
* 【14.9K】https://github.com/JeffLi1993/springboot-learning-example
* 【4.5K】https://github.com/qibaoguang/Spring-Boot-Reference-Guide
* 【7.4K】https://github.com/zhoutaoo/SpringCloud
* 【30.6K】https://github.com/eugenp/tutorials
* 【3.9K】https://github.com/ityouknow/awesome-spring-boot
* 【4.3K】https://github.com/hansonwang99/Spring-Boot-In-Action
* 【0.8K】https://github.com/cicadasmile/middle-ware-parent
* 【1.5K】https://github.com/yzcheng90/X-SpringBoot
* 【4.5K】https://github.com/spring-projects/spring-data-examples
* 【2.0K】https://github.com/vector4wang/spring-boot-quick
* 【2.2K】https://github.com/qq53182347/liugh-parent
* 【3.1K】https://github.com/lenve/JavaEETest
* 【2.1K】https://github.com/forezp/SpringBootLearning
* 【4.3K】https://github.com/zuihou/lamp-cloud
* 【2.4K】https://github.com/yizhiwazi/springboot-socks

### 3. 其他学习站点

* https://www.baeldung.com/full_archive
* https://www.petrikainulainen.net/tutorials/
