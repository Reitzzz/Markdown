# 1. 使用file模式部署

## 目录

- [1. 使用file模式部署](#1-使用file模式部署)
  - [目录](#目录)
  - [1.1 下载与服务端启动](#11-下载与服务端启动)
  - [1.2 事务分组机制与集群映射](#12-事务分组机制与集群映射)
  - [1.3 客户端配置与依赖注入](#13-客户端配置与依赖注入)
  - [1.4 开启分布式事务代理](#14-开启分布式事务代理)
  - [1.5 业务方法添加全局事务注解](#15-业务方法添加全局事务注解)
  - [1.6 创建 undo\_log 回滚日志表](#16-创建-undo_log-回滚日志表)
  - [1.7 异常回滚测试与验证](#17-异常回滚测试与验证)

---

## 1.1 下载与服务端启动

Seata也是以服务端形式进行部署的，然后每个服务都是客户端，服务端下载地址：https://github.com/seata/seata/releases/download/v1.4.2/seata-server-1.4.2.zip

把源码也下载一下：https://github.com/seata/seata/archive/refs/heads/develop.zip

下载完成之后，放入到IDEA项目目录中，添加启动配置，这里端口使用8868：

![image-20230306233336803](https://s2.loli.net/2023/03/06/ykH1BSPcxlvY4on.png)

## 1.2 事务分组机制与集群映射

Seata服务端支持本地部署或是基于注册发现中心部署（比如Nacos、Eureka等），这里我们首先演示一下最简单的本地部署，不需要对Seata的配置文件做任何修改。

Seata存在着事务分组机制：

- 事务分组：seata的资源逻辑，可以按微服务的需要，在应用程序（客户端）对自行定义事务分组，每组取一个名字。
- 集群：seata-server服务端一个或多个节点组成的集群cluster。 应用程序（客户端）使用时需要指定事务逻辑分组与Seata服务端集群（默认为default）的映射关系。

为啥要设计成通过事务分组再直接映射到集群？干嘛不直接指定集群呢？获取事务分组到映射集群的配置。这样设计后，事务分组可以作为资源的逻辑隔离单位，出现某集群故障时可以快速failover，只切换对应分组，可以把故障缩减到服务级别，但前提也是你有足够server集群。

## 1.3 客户端配置与依赖注入

接着我们需要将我们的各个服务作为Seate的客户端，只需要导入依赖即可：
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

然后添加配置：

```yaml
seata:
  service:
    vgroup-mapping:
      # 这里需要对事务组做映射，默认的分组名为 应用名称-seata-service-group，将其映射到default集群
      # 这个很关键，一定要配置对，不然会找不到服务
      bookservice-seata-service-group: default
    grouplist:
      default: localhost:8868
```

这样就可以直接启动了，但是注意现在只是单纯地连接上，并没有开启任何的分布式事务。

## 1.4 开启分布式事务代理

现在我们接着来配置开启分布式事务，首先在启动类添加注解，此注解会添加一个后置处理器将数据源封装为支持分布式事务的代理数据源（官方表示配置文件中已经默认开启了自动代理，但是UP主实测1.4.2版本下只能打注解的方式才能生效）：
```java
@EnableAutoDataSourceProxy
//@SpringBootApplication  高版本Seata已经不需要该注解
public class BookApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookApplication.class, args);
    }
}
```

## 1.5 业务方法添加全局事务注解

接着我们需要在开启分布式事务的方法上添加`@GlobalTransactional`注解：
```java
@GlobalTransactional
@Override
public boolean doBorrow(int uid, int bid) {
    //这里打印一下XID看看，其他的服务业添加这样一个打印，如果一会都打印的是同一个XID，表示使用的就是同一个事务
    System.out.println(RootContext.getXID());
    if(bookClient.bookRemain(bid) < 1)
        throw new RuntimeException("图书数量不足");
    if(userClient.userRemain(uid) < 1)
        throw new RuntimeException("用户借阅量不足");
    if(!bookClient.bookBorrow(bid))
        throw new RuntimeException("在借阅图书时出现错误！");
    if(mapper.getBorrow(uid, bid) != null)
        throw new RuntimeException("此书籍已经被此用户借阅了！");
    if(mapper.addBorrow(uid, bid) <= 0)
        throw new RuntimeException("在录入借阅信息时出现错误！");
    if(!userClient.userBorrow(uid))
        throw new RuntimeException("在借阅时出现错误！");
    return true;
}
```

## 1.6 创建 undo_log 回滚日志表

还没结束，我们前面说了，Seata会分析修改数据的sql，同时生成对应的反向回滚SQL，这个回滚记录会存放在undo_log 表中。所以要求每一个Client 都有一个对应的undo_log表（也就是说每个服务连接的数据库都需要创建这样一个表，这里由于我们三个服务都用的同一个数据库，所以说就只用在这个数据库中创建undo_log表即可），表SQL定义如下：
```sql
CREATE TABLE `undo_log`
(
  `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT,
  `branch_id`     BIGINT(20)   NOT NULL,
  `xid`           VARCHAR(100) NOT NULL,
  `context`       VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB     NOT NULL,
  `log_status`    INT(11)      NOT NULL,
  `log_created`   DATETIME     NOT NULL,
  `log_modified`  DATETIME     NOT NULL,
  `ext`           VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
```

## 1.7 异常回滚测试与验证

创建完成之后，我们现在就可以启动三个服务了，我们来测试一下当出现异常的时候是不是会正常回滚：

![image-20230306233351571](https://s2.loli.net/2023/03/06/NIe9QFW3jf1DdnV.png)

![image-20230306233359914](https://s2.loli.net/2023/03/06/LwcdO2HuWAhFr5p.png)

首先第一次肯定是正常完成借阅操作的，接着我们再次进行请求，肯定会出现异常：

![image-20230306233408870](https://s2.loli.net/2023/03/06/6VOfsp9UxYJzgKD.png)

![image-20230306233417576](https://s2.loli.net/2023/03/06/yEQa2qeiNc5npV9.png)

如果能在栈追踪信息中看到seata相关的包，那么说明分布式事务已经开始工作了，通过日志我们可以看到，出现了回滚操作：

![image-20230306233428386](https://s2.loli.net/2023/03/06/VtBlx4U1TzcqKra.png)

并且数据库中确实是回滚了扣除操作：

![image-20230306233436382](https://s2.loli.net/2023/03/06/WXn9UPgxBVhdHmb.png)

这样，我们就通过Seata简单地实现了分布式事务。