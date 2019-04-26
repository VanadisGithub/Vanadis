# Service
## 负载均衡
### CDN
### 反向代理
### 负载均衡
* 软负载
    * Nginx
        * 是WEB服务器，缓存服务器，又是反向代理服务器，可以做七层的转发
    * HAproxy
        * 基于四层和七层的转发，是专业的代理服务器
    * LVS
        * 基于四层的转发
    * 配合
        * Keepalived
            * 类似交换机，作用3、4、5层，自动切换服务
    * 阿里SLB
* 硬负载
    * F5（F5-BIG-IP-GTM）
    * Array
## 消息服务
## 应用服务
### 分布式（微服务）
* 配置管理
    * Config
    * Bus
        * 消息总线
            * 推送配置
* 服务管理（注册和发现）
    * Dubbo
    * Eureka（Netflix）
        * 只是维护了服务生产者、注册中心、服务消费者三者
    * Consul（Apache）
    * Edas
* 服务网关
    * Gateway
    * Zuul（Netflix）
        * 服务的权限控制
* 服务治理
    * 服务熔断
        * Hystrix
    * 服务降级
        * Hystrix
        * 开关预置、配置中心
    * 服务限流
        * 应用级限流
            * Tomcat
                * Connector
                    * acceptCount：如果Tomcat的线程都忙于响应，新来的连接会进入队列排队，如果超出排队大小，则拒绝连接
                    * maxConnections：瞬时最大连接数，超出的会排队等待
                    * maxThreads：Tomcat能启动用来处理请求的最大线程数，如果请求处理量一直远远大于最大线程数则可能会僵死
            * MySQL
                * max_connections
            * Redis
                * tcp-backlog
        * 分布式限流
            * redis+lua
            * nginx+lua
        * 限流算法
            * 漏桶
            * 令牌桶
* 服务生产和消费
    * Ribbon（Netflix）
    * Fegin（Netflix）
* 服务开发
    * 框架（Freamework）
        * Spring
            * Bean
                * 作用域
                 * > 作用域限定了Spring Bean的作用范围，在Spring配置文件定义Bean时，通过声明scope配置项，可以灵活定义Bean的作用范围。例如，当你希望每次IOC容器返回的Bean是同一个实例时，可以设置scope为singleton；当你希望每次IOC容器返回的Bean实例是一个新的实例时，可以设置scope为prototype。
                 * > 
                 * > scope配置项有5个属性，用于描述不同的作用域。
                 * > 
                 * > ① singleton
                 * > 
                 * > 使用该属性定义Bean时，IOC容器仅创建一个Bean实例，IOC容器每次返回的是同一个Bean实例。
                 * > 
                 * > ② prototype
                 * > 
                 * > 使用该属性定义Bean时，IOC容器可以创建多个Bean实例，每次返回的都是一个新的实例。
                 * > 
                 * > ③ request
                 * > 
                 * > 该属性仅对HTTP请求产生作用，使用该属性定义Bean时，每次HTTP请求都会创建一个新的Bean，适用于WebApplicationContext环境。
                 * > 
                 * > ④ session
                 * > 
                 * > 该属性仅用于HTTP Session，同一个Session共享一个Bean实例。不同Session使用不同的实例。
                 * > 
                 * > ⑤ global-session
                 * > 
                 * > 该属性仅用于HTTP Session，同session作用域不同的是，所有的Session共享一个Bean实例。

                    * singleton
                        * 线程不安全
                    * prototzpe
                    * request
                    * session
                    * global session
                * 生命周期
                 * > Spring的生命周期.
                 * > 
                 * > 1.       容器启动，实例化所有实现了BeanFactoyPostProcessor接口的类。他会在任何普通Bean实例化之前加载.
                 * > 
                 * > 2.       实例化剩下的Bean，对这些Bean进行依赖注入。
                 * > 
                 * > 3.       如果Bean有实现BeanNameAware的接口那么对这些Bean进行调用
                 * > 
                 * > 4.       如果Bean有实现BeanFactoryAware接口的那么对这些Bean进行调用
                 * > 
                 * > 5.       如果Bean有实现ApplicationContextAware接口的那么对这些Bean进行调用
                 * > 
                 * > 6.       如果配置有实现BeanPostProcessor的Bean，那么调用它的postProcessBeforeInitialization方法
                 * > 
                 * > 7.       如果Bean有实现InitializingBean接口那么对这些Bean进行调用
                 * > 
                 * > 8.       如果Bean配置有init属性，那么调用它属性中设置的方法
                 * > 
                 * > 9.       如果配置有实现BeanPostProcessor的Bean，那么调用它的postProcessAfterInitialization方法
                 * > 
                 * > 10.   Bean正常是使用
                 * > 
                 * > 11.   调用DisposableBean接口的destory方法
                 * > 
                 * > 12.   调用Bean定义的destory方法
                 * > 
                 * >  
                 * > 
                 * > 如果从大体上区分值分只为四个阶段
                 * > 
                 * > 1.       BeanFactoyPostProcessor实例化
                 * > 
                 * > 2.       Bean实例化，然后通过某些BeanFactoyPostProcessor来进行依赖注入
                 * > 
                 * > 3.       BeanPostProcessor的调用.Spring内置的BeanPostProcessor负责调用Bean实现的接口: BeanNameAware, BeanFactoryAware, ApplicationContextAware等等，等这些内置的BeanPostProcessor调用完后才会调用自己配置的BeanPostProcessor
                 * > 
                 * > 4.       Bean销毁阶段

                * 结构：BeanDefinition
                 * > 可以看到上面的很多属性和方法都很熟悉，例如类名、scope、属性、构造函数参数列表、依赖的bean、是否是单例类、是否是懒加载等，其实就是将Bean的定义信息存储到这个BeanDefinition相应的属性中，后面对Bean的操作就直接对BeanDefinition进行，例如拿到这个BeanDefinition后，可以根据里面的类名、构造函数、构造函数参数，使用反射进行对象创建。
                 * > 
                 * > BeanDefinition是一个接口，是一个抽象的定义，实际使用的是其实现类，如ChildBeanDefinition、RootBeanDefinition、GenericBeanDefinition等。
                 * > 
                 * > BeanDefinition继承了AttributeAccessor，说明它具有处理属性的能力；
                 * > 
                 * > BeanDefinition继承了BeanMetadataElement，说明它可以持有Bean元数据元素，作用是可以持有XML文件的一个bean标签对应的Object。

            * IOC
             * > 1.降低组件之间的耦合度，实现软件各层之间的解耦. 
             * > 
             * > 2.可以使容器提供众多服务如事务管理消息服务处理等等。当我们使用容器管理事务时，开发人员就不需要手工 控制事务，也不需要处理复杂的事务传播
             * > 
             * >  3.容器提供单例模式支持，开发人员不需要自己编写实现代码.
             * > 
             * >  4.容器提供了AOP技术，利用它很容易实现如权限拦截，运行期监控等功能 
             * > 
             * > 5.容器提供众多的辅佐类，使这些类可以加快应用的开发.如jdbcTemplate HibernateTemplate

                * BeanFactory 接口
                * ApplicationContext 接口
                * DI 依赖注入
                    * 注入方式
                        * set方式注入
                        * 构造方法注入
                        * 字段注入
                    * 注入类型
                        * 值类型注入
                        * 引用类型注入
            * AOP
            * 事务
                * 编程式事务管理
                * 声明式事务管理
        * SpringBoot
            * 注解
                * @SpringBootApplication
                    * @SpringBootConfiguration
                    * @EnableAutoConfiguration
                        * EnableAutoConfigurationImportSelector.class
                            * AutoConfigurationImportSelector.java
                                * SpringFactoriesLoader
                                    * 从指定的配置文件META-INF/spring.factories中加载配置
                    * @ComponentScan
                * @RestController
            * 组件
                * Data
                * Security
                * Actuator
            * 配置
                * bootstrap
                 * > bootstrap 配置文件有以下几个应用场景。
                 * > 
                 * > 使用 Spring Cloud Config 配置中心时，这时需要在 bootstrap 配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；
                 * > 
                 * > 一些固定的不能被覆盖的属性
                 * > 
                 * > 一些加密/解密的场景；

                    * 优先加载
                    * 不能覆盖
                * application
        * Play
            * 全栈框架
            * 热部署
    * 工具
        * Maven
            * 依赖
                * 依赖传递
                * 最近依赖
                * 继承与聚合
            * 命令
                * deploy
                    * Nexus
        * Git
        * Jenkins
        * Sonar
    * 辅助
        * Swagger
        * Pandora
* 服务通信
    * 服务间通信
        * RPC
            * dubbo
            * webservice
            * hessian
            * http
            * RMI
        * Netty
    * 消息队列
        * Kafka
        * ActiveMQ
        * RabbitMQ
        * RocketMQ（阿里）
        * QSP
* 服务部署
    * 容器
        * Docker
            * Swarm
            * Compose
            * Machine
        * Kubernetes
### 集群
* LB：Load Banlancing（负载）
* HA：High Availability（高可用）
* HP：High Performcace（高性能）
## 数据服务
### 缓存
* Memcache
* Redis
    * 缓存穿透
        * 对null数据缓存较短时间
    * 缓存击穿
        * 热点数据不过期
    * 缓存雪崩
        * 数据预热，随机缓存时间
* Varish
### 数据库
* 关系型
    * MySql
        * 索引
            * 存储类型
                * B 树
                * Hash
            * 分类
                * 普通索引
                * 唯一索引
                * 单列索引
                * 组合索引
                * 全文索引
                    * MySQL中只有MyISAM存储引擎支持全文索引
        * 引擎
            * InnoDB
            * MyISAM
        * 查询优化
            * 使用 Explain 进行分析
            * 优化数据访问
                * 1. 减少请求的数据量
                * 2. 减少服务器端扫描的行数
            * 重构查询方式
                * 1. 切分大查询
                * 2. 分解大连接查询
        * 数据库优化
            * 主从热备
                * Mysql:master-slave
                * 实现
                    * binlog 线程 ：负责将主服务器上的数据更改写入二进制日志（Binary log）中。
                    * I/O 线程 ：负责从主服务器上读取二进制日志，并写入从服务器的中继日志（Relay log）。
                    * SQL 线程 ：负责读取中继日志，解析出主服务器已经执行的数据更改并在从服务器中执行。
            * 读写分离
                * 应用层/中间件
            * 分库分表
                * 条件
                    * 单表>500万条
                    * 单表>2G
                * 工具
                    * MyCat
                    * Cobar
        * 事务
        * 锁
    * ADS
    * RDS
* 非关系型
    * MongoDB
        * 文档模型
        * 分片
    * Redis
### 数据库中间件
* DRDS（阿里）
* MyCat
* Cobar
### 大数据
* MaxCompute（原ODPS）
* Hadoop
* Hbase
* Spark
* Storm/JStorm
### 搜索引擎
* EasticSerach
* Solr
### 文件存储
* OSS（阿里）