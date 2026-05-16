# 1 使用 Nacos 模式部署

- [1 使用 Nacos 模式部署](#1-使用-nacos-模式部署)
  - [1.1 服务端 Nacos 注册中心配置](#11-服务端-nacos-注册中心配置)
  - [1.2 配置文件导入与热更新管理](#12-配置文件导入与热更新管理)
  - [1.3 服务端事务组映射配置](#13-服务端事务组映射配置)
  - [1.4 客户端 Nacos 配置接入](#14-客户端-nacos-配置接入)
  - [1.5 配置数据库作为会话存储](#15-配置数据库作为会话存储)
  - [1.6 高并发场景下的思考](#16-高并发场景下的思考)

---

## 1.1 服务端 Nacos 注册中心配置

前面我们实现了本地 Seata 服务的 file 模式部署，现在我们来看看如何让其配合 Nacos 进行部署，利用 Nacos 的配置管理和服务发现机制，Seata 能够更好地工作。

我们先单独为 Seata 配置一个命名空间：

![image-20230306233444767](https://s2.loli.net/2023/03/06/93mXN5dlC2GTLOW.png)

我们打开 `conf` 目录中的 `registry.conf` 配置文件：
```properties
registry {
  # 注册配置
  # 可以看到这里可以选择类型，默认情况下是普通的file类型，也就是本地文件的形式进行注册配置
  # 支持的类型如下，对应的类型在下面都有对应的配置
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  # 采用nacos方式会将seata服务端也注册到nacos中，这样客户端就可以利用服务发现自动找到seata服务
  # 就不需要我们手动指定IP和端口了，不过看似方便，坑倒是不少，后面再说
  nacos {
    # 应用名称，这里默认就行
    application = "seata-server"
    # Nacos服务器地址
    serverAddr = "localhost:8848"
    # 这里使用的是SEATA_GROUP组，一会注册到Nacos中就是这个组
    group = "SEATA_GROUP"
    # 这里就使用我们上面单独为seata配置的命名空间，注意填的是ID
    namespace = "89fc2145-4676-48b8-9edd-29e867879bcb"
    # 集群名称，这里还是使用default
    cluster = "default"
    # Nacos的用户名和密码
    username = "nacos"
    password = "nacos"
  }
    #...
```

## 1.2 配置文件导入与热更新管理

注册信息配置完成之后，接着我们需要将配置文件也放到 Nacos 中，让 Nacos 管理配置，这样我们就可以对配置进行热更新了，一旦环境需要变化，只需要直接在 Nacos 中修改即可。
```properties
config {
  # 这里我们也使用nacos
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    # 跟上面一样的配法
    serverAddr = "127.0.0.1:8848"
    namespace = "89fc2145-4676-48b8-9edd-29e867879bcb"
    group = "SEATA_GROUP"
    username = "nacos"
    password = "nacos"
    # 这个不用改，默认就行
    dataId = "seataServer.properties"
  }
```

接着，我们需要将配置导入到 Nacos 中，我们打开一开始下载的源码 `script/config-center/nacos` 目录，这是官方提供的上传脚本，我们直接运行即可（windows下没对应的bat就很蛋疼，可以使用git命令行来运行一下），这里我们使用这个可交互的版本：

![image-20230306233500474](https://s2.loli.net/2023/03/06/1tPwBFn7u3ScCeY.png)

按照提示输入就可以了，不输入就使用的默认值，不知道为啥最新版本有四个因为参数过长还导入失败了，就离谱，不过不影响。

导入成功之后，可以在对应的命名空间下看到对应的配置（为啥非要一个一个配置项单独搞，就不能写一起吗）：

![image-20230306233510973](https://s2.loli.net/2023/03/06/8yTQGZluYVe1cg2.png)

## 1.3 服务端事务组映射配置

注意，还没完，我们还需要将对应的事务组映射配置也添加上，DataId 格式为 `service.vgroupMapping.事务组名称`，比如我们就使用默认的名称，值全部依然使用 default 即可：

![image-20230306233521002](https://s2.loli.net/2023/03/06/UBchb4zPjHAfCSs.png)

## 1.4 客户端 Nacos 配置接入

现在我们就完成了服务端的 Nacos 配置，接着我们需要对客户端也进行 Nacos 配置：
```yaml
seata:
  # 注册
  registry:
    # 使用Nacos
    type: nacos
    nacos:
      # 使用Seata的命名空间，这样才能正确找到Seata服务，由于组使用的是SEATA_GROUP，配置默认值就是，就不用配了
      namespace: 89fc2145-4676-48b8-9edd-29e867879bcb
      username: nacos
      password: nacos
  # 配置
  config:
    type: nacos
    nacos:
      namespace: 89fc2145-4676-48b8-9edd-29e867879bcb
      username: nacos
      password: nacos
```

现在我们就可以启动这三个服务了，可以在 Nacos 中看到 Seata 以及三个服务都正常注册了：

![image-20230306233529255](https://s2.loli.net/2023/03/06/PSbw5TFhm74Wu3n.png)

![image-20230306233538630](https://s2.loli.net/2023/03/06/nZUcuM2kJ86zgBv.png)

接着我们就可以访问一下服务试试看了：

![image-20230306233545257](https://s2.loli.net/2023/03/06/Fn3R2Jrq1YyleCh.png)

可以看到效果和上面是一样的，不过现在我们的注册和配置都继承在 Nacos 中进行了。

## 1.5 配置数据库作为会话存储

我们还可以配置一下事务会话信息的存储方式，默认是 file 类型，那么就会在运行目录下创建 `file_store` 目录，我们可以将其搬到数据库中存储，只需要修改一下配置即可：

![image-20230306233553931](https://s2.loli.net/2023/03/06/Cph9zPF2kaSvKdY.png)

将 `store.session.mode` 和 `store.mode` 的值修改为 `db`

接着我们对数据库信息进行一下配置：

* 数据库驱动
* 数据库 URL
* 数据库用户名密码

其他的默认即可：

![image-20230306233612224](https://s2.loli.net/2023/03/06/dlmYNnARZaxJ5MH.png)

接着我们需要将对应的数据库进行创建，创建 seata 数据库，然后直接 CV 以下语句：
```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_status_gmt_modified` (`status` , `gmt_modified`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `status`         TINYINT      NOT NULL DEFAULT '0' COMMENT '0:locked ,1:rollbacking',
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_status` (`status`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

CREATE TABLE IF NOT EXISTS `distributed_lock`
(
    `lock_key`       CHAR(20) NOT NULL,
    `lock_value`     VARCHAR(20) NOT NULL,
    `expire`         BIGINT,
    primary key (`lock_key`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4;

INSERT INTO `distributed_lock` (lock_key, lock_value, expire) VALUES ('HandleAllSession', ' ', 0);
```

![image-20230306233627086](https://s2.loli.net/2023/03/06/7zvewSLhFmbc8G1.png)

完成之后，重启 Seata 服务端即可：

![image-20230306233752098](https://s2.loli.net/2023/03/06/G7qQoEy8DCX9bLJ.png)

看到了数据源初始化成功，现在已经在使用数据库进行会话存储了。

如果 Seata 服务端出现报错，可能是我们自定义事务组的名称太长了：

![image-20230306233933641](https://s2.loli.net/2023/03/06/qoNhgzM2PXpZU9B.png)

将 `globle_table` 表的字段 `transaction_server_group` 长度适当增加一下即可：

![image-20230306233940850](https://s2.loli.net/2023/03/06/9LnaoUxHzlY1GdV.png)

到此，关于基于 nacos 模式下的 Seata 部署，就完成了。

## 1.6 高并发场景下的思考

虽然我们这里实现了分布式事务，但是还是给各位同学提出一个问题（可以把自己所认为的结果打在弹幕上），就我们目前这样的程序设计，在高并发下，真的安全吗？比如同一时间 100 个同学抢同一个书，但是我们知道同一个书就只有 3 本，如果这时真的同时来了 100 个请求要借书，会正常地只借出 3 本书吗？如果不正常，该如何处理？