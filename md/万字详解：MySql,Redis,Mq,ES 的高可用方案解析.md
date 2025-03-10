> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7126864114806177822)

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第 2 天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e30bbd776a6249ea91b910d8fe702194~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

> 大家好我是一灰灰，本文将接着前文 [1w5 字详细介绍分布式系统的那些技术方案](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU3MTAzNTMzMQ%3D%3D%26mid%3D2247487507%26idx%3D1%26sn%3D9c4ff02747e8335ee5e3c7765cc80b3c%26chksm%3Dfce70bbfcb9082a9a8d972af80f19a9b66a5425c949bc400872727cc2da9f401047a5a523ac4%26token%3D309565785%26lang%3Dzh_CN%23rd "https://mp.weixin.qq.com/s?__biz=MzU3MTAzNTMzMQ==&mid=2247487507&idx=1&sn=9c4ff02747e8335ee5e3c7765cc80b3c&chksm=fce70bbfcb9082a9a8d972af80f19a9b66a5425c949bc400872727cc2da9f401047a5a523ac4&token=309565785&lang=zh_CN#rd") 文章基础上，进行实际的案例解析

高可用对于当下的系统而言，可以说是一个硬指标，常年专注于业务开发的我们，对于高可用最直观的感觉可能就是祈祷应用不要出问题，不要报错；即便有问题，也最好不是我们的业务代码逻辑导致的，如果是服务器、DB、中间件 (如注册中心、配置中心等) 的异常那就抛给对应的 sre, dba；然而常在河边走，哪有不湿鞋，为了保障服务的高可用，我们可以从哪些方面进行努力呢？

本文将作为高可用的开篇，通过简述一些常用的系统的高可用方案，给大家介绍一下我们可以从哪些方面努力让我们的系统达到高可用，主要设计到的系统如下

*   缓存：Redis
*   数据库：MySql
*   消息队列：RabbitMQ
*   搜索: ElasticSearch

1 redis 高可用策略
-------------

redis 广泛应用于缓存的业务场景，当然也有将其当做持久化存储的 nosql 数据库使用，这些都不重要，重点是 redis 在提供服务的时候，是如何支持高可用的呢？

redis 官方支持了四种策略：

*   数据持久化
*   主从同步
*   哨兵模式
*   集群

除以上姿势之外，我们自己在使用时还可以选择根据业务场景使用不同的 redis 实例（即传说中的不把所有鸡蛋放在一个篮子里）

接下来将针对 redis 的几种高可用策略进行简述说明

### 1.1 数据持久化

> 官方手册: [Redis persistence](https://link.juejin.cn?target=https%3A%2F%2Fredis.io%2Fdocs%2Fmanual%2Fpersistence%2F "https://redis.io/docs/manual/persistence/")

持久化是在高可用、一致性的场景中经常会看到的一种技术手段；

在高可用的场景中，数据的持久化主要是为了解决在服务出现问题（如宕机）之后，可以快速恢复并对外继续提供服务能力；

redis 官方提供了两种持久化策略

*   AOF: 将更新的操作命令记录在对应的日志文件中，在重启的时候采用 “回放” 策略，将所有的命令重新执行一遍来实现场景恢复
*   RDB: 定时存储 redis 中的数据快照到数据文件中，在重启的时候，加载 rdb 文件，恢复所有的数据

简单来讲 AOF 记录的是操作动作，采用回放执行的机制进行恢复；RDB 则相当于数据落盘，重新读取加载的机制进行恢复

**注：AOF RDB 可以一起工作，没有排他性**

### 1.2 主从方式

虽然 redis 性能爆炸，但是单机依然存在性能瓶颈；当我们遇到单机的性能瓶颈的时候，一般怎么做？

没错，加机器

redis 也支持多机服务，比如常见的一主多从策略：

*   主机：提供读写能力
*   从机：只提供读

针对绝大多数读多写少的场景，我们可以起多个 redis 实例，其中一个设置为主，提供所有的写请求；其他的实例则设置为从，客户端通过负载策略路由到不同的从 redis，从而实现流量分摊；

同时也因为有多个实例，所以单台或几台实例下线，对整个服务的可用性影响并不会太大（及时摘除故障机器，其他的实例依然可以正常提供服务；当然前提是流量所示太大把其他的实例也打挂，那就 gg 了）

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

主从模式还有一个变种，叫做从从模式，主要是为了解决主 redis 的同步压力，改成主 -> 从，然后由一个从同步给其他的从实例，具体架构图如下

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

使用主从、主从从模式实现高可用可算是分布式系统的经典策略，其主要思想在于：

*   多实例提供服务，实现负载均衡
*   每个实例**冗余**一份全量数据

### 1.3 哨兵模式

> 官方手册: [High availability with Redis Sentinel](https://link.juejin.cn?target=https%3A%2F%2Fredis.io%2Fdocs%2Fmanual%2Fsentinel%2F "https://redis.io/docs/manual/sentinel/")

哨兵模式主要是为了解决主从模式中，主机宕机的场景，由于主机本身存在单点，所以主节点对成了高可用的关键因素了；那么如果实现主节点宕机之后，自动选择一个新的主节点，这样不就可以提高系统的可用性了么； redis 官方提供的机制就是 - 哨兵模式

主要工作原理：

*   哨兵：监听 redis 实例，判断是否存活（不太对外提供服务能力）
*   通过 PING 命令，检查与主从服务器之间的连接情况，若正常相应，则认为存活；否则认为`主观下线`
*   当 `n/2 + 1`半数以上哨兵认为主节点下线，则认为主节点`客观下线`，尝试选新的主节点
*   从所有从节点中，选择与之前主库相似度最高的从节点作为新的主库

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

哨兵模式，可以理解为探活 + 选主，而这也常见于各大分布式系统的技术方案中

### 1.4 集群模式

> 官方手册: [Scaling with Redis Cluster](https://link.juejin.cn?target=https%3A%2F%2Fredis.io%2Fdocs%2Fmanual%2Fscaling%2F "https://redis.io/docs/manual/scaling/")

相比于主从模式的全量冗余，redis 的集群策略在在于数据分片，每个实例上存储部分的数据；而不是全量数据，从而解决数据量大的场景下，对于 redis 服务本身以及数据同步的压力

集群模式的特点在于多个实例，构建成一个实例，每个实例上存储部分的数据；redis 并没有采用一致性 hash 来做数据分布，而是使用特有的 slots 插槽机制，来实现数据的 hash 映射

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

集群模式，主要特点在于数据分片，每个实例存部分数据，其思路在于**拆分**

从上面的图中也可以看出，集群一般与主从搭配使用，集群中的每个分片对应的是主从模式的 redis 服务，从而加强高可用

### 1.5 小结

这一节主要介绍的是 redis 的高可用策略，从中也可以看到很多经典的技术方案

*   持久化：RDB 数据落盘加载方式 + AOF 记录操作命令用于回放策略
*   主从，主从从：全量数据冗余、读写请求分离，负载均衡的思想；核心问题在于主节点挂掉之后需要人工参与手动指定主库
*   哨兵机制：PING/PONG 的探活机制，监听主节点，宕机之后自动选主，确保高可用；核心问题在于所有的实例冗余相同的一份数据，数据量大时不友好
*   集群：数据分片，每个实例提供部分服务能力

看到这里的小伙伴自然会想到，为什么 redis 会提供这些不同的策略？它们各自的应用场景是什么，优缺点是啥？这些疑问就放在后续的 redis 高可用详解中介绍

相关博文：

*   [Scaling with Redis Cluster | Redis](https://link.juejin.cn?target=https%3A%2F%2Fredis.io%2Fdocs%2Fmanual%2Fscaling%2F "https://redis.io/docs/manual/scaling/")
*   [Redis 高可用策略 - 楼仔](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FgofXUXKD_ZelbOJDinHs4g "https://mp.weixin.qq.com/s/gofXUXKD_ZelbOJDinHs4g")
*   [redis 系列之——高可用（主从、哨兵、集群）诸葛小猿](https://link.juejin.cn?target=https%3A%2F%2Fxie.infoq.cn%2Farticle%2F6c3500c66c3cdee3d72b88780 "https://xie.infoq.cn/article/6c3500c66c3cdee3d72b88780")

2 MySql 高可用策略
-------------

MySql 数据库的高可用策略就比较多了，同样也非常的经典；仅仅主节点的保活策略就非常多了；在这里将主要的重心放在 MySql 的高可用架构主备、主从、一主多从，多主多从上，至于主节点故障时转移策略则放在后续详细的文章中进行介绍

### 2.1 数据持久化

对于每个开发者而言，大多都听说过数据库的 ACID 特性，其中的 D 对应的就是这里说到的持久化；区别于 redis 的持久化，以 MySql 的 InnoDB 引擎为例，其持久化涉及到多个日志文件 (undo log,redo log,binlog)，缓存区 (buffer)，磁盘 (idb 文件)

接下来看一下完整的数据更新 / 插入的流程

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

接下来描述一下核心思想：

*   数据更新策略：总是更新缓存的内容（缓存未命中，则从磁盘加载到缓存）
*   先写 undolog 日志文件：记录之前的数据，支持 mvcc、支持回滚就靠它
*   redolog 记录的两阶段提交：（先是 prepare，待 binlog 写完之后，再次更新状态为 commit）
*   最后异步刷新缓存数据到磁盘

虽然上面的描述比较简单，但是这里的知识点非常多，如

*   为什么先更新缓存，最后异步刷磁盘？
    *   核心在于操作内存的速度 >> 操作磁盘
*   undolog 作用是什么，怎么支持 mvcc，实现事务回滚的？
    *   保障事务原子性的关键所在，数据行非主键变更时，记录修改前的数据到 undolog，并指向它，其他 sql 读这个 undolog 中的副本数据从而支持 mvcc，回滚时则是根据 undo log 进行逻辑恢复
*   redolog 作用是什么，为什么两阶段方案？
    *   主要保障事务的持久性，当数据库异常宕机之后，可以通过重新执行 redo log 来恢复未及时落盘的数据；两阶段的主要目的是为了解决 redolog 与 binlog 的一致性问题，避免出现 redolog 第一阶段成功，但是 binlog 失败导致不一致问题
    *   redolog 属于 innodb 引擎，固定大小，环形结构覆盖写策略；内部同样是先写缓存，再刷磁盘的策略

更多详情内容，后面到 mysql 的专题时再详细介绍

### 2.2 主备架构

保证高可用的一个最简单策略就是 “冗余”，也就是我们这里说到的主备架构，对 mysql 而言，就是我启动两个实例；一个主库对外提供读写服务，一个备库，冗余主库的所有数据内容，并不对外提供服务；

当主库 gg 之后，然后备库升级，切换为主库

> 话说这个思想和古代的储君制非常像了，平时都是皇帝总领朝堂，太子就当吉祥物；皇帝驾崩之后，太子就晋升为皇帝（论备胎的重要性）

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

主备的最大特点就是多备一台实例，在出问题时顶上，当然缺点就很明显了，严重的资源浪费

### 2.3 主从架构

主从和前面 mysql 的思路差不多，主从模式一般又叫做读写分离，即写主库，读从库；相比于主备而言，最主要的突破点在于另外一个 mysql 实例不会干放着，而是尤其来承担读请求

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/86115e9b610d4bfebdf49e5aed1e9a6a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

主从的核心思想在于读写分离

### 2.4 一主多从

在前面主从的基础上多挂几个从库，主要出发点在于当前的互联网场景下，绝大多数的应用都是读多写少，通过挂多个从库，可以有效提供整体服务的性能指标

同样一主多从的模式，也会区分为主从 + 主从从两种，后者则主要是为了减少主库的同步压力，下图为核心 4 架构模型

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

### 2.5 多主多从

一主多从可以解决读多写少的场景，但总会出现写瓶颈的场景；在不考虑分库分表的业务手段之前（这种方式也可以理解为数据分片，类似上面说到的 redis 集群模式），仅仅从 mysql 的架构模式出发，自然会想到的策略就是多个主库提供写能力，这就是我们说的多主多从的架构了

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

多主多从，其中每个主库都可以独立对外提供写请求；从库则对外提供读请求

需要注意的是主库之间的数据同步，即一个写请求落到一个任意一个主库之后，所有的主库都会同步这个写操作

### 2.6 主库切换策略、主从同步策略

前面介绍的是几种不同的主从架构特点，主要通过主、备 / 从来新增实例来提高可用性；但是还有两个非常重要的点没有细说，一个是故障之后，如何确定新的主库；另外一个则是主从 / 主主之间的数据如何同步，如何保证数据的一致性；

接下来我们将简单的介绍下 mysql 中常见的一些做法（更详细的当然留在后面的专题）

#### 主库切换策略

**VIP + KeepAlived**

*   vip: 即 virtual ip 虚拟 ip
*   KeepAlived: 保活脚本

其主要思路在于外部通过 VIP 访问 mysql 实例 (主从 / 主主)，而 KeepAlived 用于检测主库是否存活，当挂掉之后，VIP 偏移到另外一个主库（或者选一个从库作为主库）上，从而实现自动的切主流程

缺点：

*   级联复制 (主 -> 从 ->从这种复制模式叫做级联复制)或者一主多从在切换之后，其他从实例需要重新配置连接新主

**MHA**

Master High Avaliable 主库高可用机制，也是当下很多公司采用的策略；其包含一套完整的工具，在检测到主库不可用后，会自动将同步到最接近主库的 slave 提升为 master，然后将其他的 slave 指向新的 master

其优点非常明显，通常可以实现十秒内的主从切换，扩展 MySql 节点也非常方便；而缺点则在于主要监控主库

**MXC**

PXC（Percona XtraDB Cluster）是一个完全开源的 MySQL 高可用解决方案。它将 Percona Server、Percona XtraBackup 与 Galera 库集成在一起，以实现多主复制的 MySQL 集群

其核心特点在于写请求会自动同步到其他节点，要求在所有的节点都验证之后才会提交，保证数据的强一致性

因此缺点就在于木桶效应，性能取决于最差的那个节点

**MGR/InnoDB Cluste**

MySQL 5.7 推出了 MGR（MySQL Group Replication），与 PXC 类似，也实现了多节点数据写入和强一致性的特点。MGR 采用 GCS（Group Communication System）协议同步数据，GCS 可保证消息的原子性

外部连接通过 MySql router 与一组 mysql 实例进行交互，当主库切换时，mysql router 会自动切换到新的主节点

**Xenon**

给予 Raft 协议的 MySql 高可用和复制性管理工具，无中心化选主，支持秒级切换

#### 主从同步策略

当存在主从库时，必然会存在同步问题，如何保障主库与从库数据的一致性呢？

**主从同步流程**

主从同步主要借助 Binlog 来实现，这个在前面的图中有简单的体现，下面则是相对完整的同步流程

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

*   主库生成 binlog 日志文件
    *   statement: 记录具体引起改动的操作语句，比如 insert xxxxx，缺点是某些函数会导致数据不一致（如 now()）
    *   row: 基于数据行的，原来数据行是 xx 值改为了 yy 值，缺点是数据量大
    *   mixed: 上面两个混用
*   从库的 io 线程拉主库的 binlog 日志，写入自己的 relaylog(中继日志)，然后由 sql 线程读取 relaylog 日志进行回放，实现数据同步

**主从同步策略**

使用主从之后，在实际的业务开发中，最最常遇到的问题就是主从延迟，即主库数据已经写入了，但是读从库却读不到对应的数据，这个就是主从延迟了，它直接导致数据的不一致；当然一般这种影响还好，但是如果因为主从延迟，现在主库挂了，所有的从库都没有最新的记录，这不就导致数据丢失了么，会导致严重的数据一致性问题

所以在主从同步的策略上，有下面几种

case1: 异步复制

主库完成写请求之后，理解返回结果，并不关心从库是否同步接收处理，此时就可能出现上面说的，主库挂了之后，所有从库还存在未同步的数据，导致数据丢失

case2: 半同步复制

为了避免出现上面的问题，我们要求最少有一个从库同步完之后，才响应用户端请求，这样表明主库宕机之后还有个兜底的

case3: 全同步复制

这个更激进一点，要求所有的从库都同步完，才算真正的 ok，保证强一致性，缺点则在于性能会受到影响

### 2.7 小结

这一小节主要介绍的是 MySql 的高可用策略，从架构方面出发，有主备，主从，一主多从，多主多从，同时也简单的介绍了下如何实现主库的自动切换 (MHA,MXC,MGR 等)、主从数据同步流程，同步策略；如果想了解更详细的内容，请移步到 mysql 的高可用专题

下面小结一下保持高可用的主要思路

*   通过冗余来实现高可用：如主备
*   读写分离，实现负载均衡：主从、主从从模式
*   数据持久化策略：操作内存 (buffer)，异步刷盘，两阶段提交保障一致性

相关博文:

*   [官方文档 InnoDB Cluster](https://link.juejin.cn?target=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F8.0%2Fen%2Fmysql-innodb-cluster-introduction.html "https://dev.mysql.com/doc/refman/8.0/en/mysql-innodb-cluster-introduction.html")
*   [读完搞懂 MySQL 持久化和回滚（图文详解）-mysql 教程](https://link.juejin.cn?target=https%3A%2F%2Fwww.php.cn%2Fmysql-tutorials-488418.html "https://www.php.cn/mysql-tutorials-488418.html")
*   [MySQL 常用高可用方案](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F3ICMQUF_vQpuDm2nHi1qqw "https://mp.weixin.qq.com/s/3ICMQUF_vQpuDm2nHi1qqw")
*   [MySQL 高可用之 PXC 详解_现实如此呀的博客 - CSDN 博客_pxc](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_47019016%2Farticle%2Fdetails%2F114740096 "https://blog.csdn.net/weixin_47019016/article/details/114740096")
*   [一文看懂 MySQL 的异步复制、全同步复制与半同步复制](https://link.juejin.cn?target=https%3A%2F%2Fwww.51cto.com%2Farticle%2F606556.html "https://www.51cto.com/article/606556.html")

3. RabbitMq 高可用方案
-----------------

消息中间件也是大家或多或少会接触的一类系统，接下来将以 RabbitMq 来看一下它的高可用是如何实现的

### 3.1 数据持久化

不同于前面 MySql 必然会持久化，RabbitMq 的数据持久化是可选的，当我们对数据的完整性要求高时，最好开启持久化

首先简单看一下 rabbitmq 的模型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b41b96738db4238ad82cae65473a2f8~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们这里说的持久化主要指

*   exchange 持久化: 即 exchange 本身不会因为 rabbitmq 宕机而被删除，需要手动指定 durable=true
*   topic 持久化：消费者通过 topic 从 exchange 中读取消息，需要指定 durable=true，避免出现宕机后队列中的消息丢失
*   msg 消息持久化：即生产者投递到 echange 的消息，需要持久化到磁盘

注意 rabbitmq 的消息持久化也是先写到 buffer，然后再定时刷新到磁盘；

当我们为了保障数据的完整性时，一般会开启消息的确认机制 / 事务机制，每次投递等到 mq 回复一个确认 ack 之后，才表示真正的投递成功，而 mq 的应答则是在消息的持久化之后进行

### 3.2 主备模式

同前面的 MySql 的主备，主节点提供读写，备节点同步主节点的数据，不对外提供服务能力；当主节点挂了之后，启用备节点对外服务，原主节点恢复之后则作为备节点存在

### 3.3 Shovel 远程模式

> 官方文档： * [Shovel Plugin — RabbitMQ](https://link.juejin.cn?target=https%3A%2F%2Fwww.rabbitmq.com%2Fshovel.html "https://www.rabbitmq.com/shovel.html")

远程模式可以实现双活的一种模式，简称 shovel 模式，所谓的 shovel 就是把消息进行不同数据中心的复制工作，可以跨地域的让两个 MQ 集群互联，远距离通信和复制。

*   Shovel 就是我们可以把消息进行数据中心的复制工作，我们可以跨地域的让两个 MQ 集群互联。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5119890b44a42fc9071c3f952e425d7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

如上图，有两个异地的 MQ 集群（可以是更多的集群），当用户在地区 1 这里下单了，系统发消息到 1 区的 MQ 服务器，发现 MQ 服务已超过设定的阈值，负载过高，这条消息就会被转到 地区 2 的 MQ 服务器上，由 2 区的去执行后面的业务逻辑，相当于分摊我们的服务压力。

### 3.4 镜像模式

如下图，用 KeepAlived 做了 HA-Proxy 的高可用，然后有 3 个节点的 MQ 服务，消息发送到主节点上，主节点通过 mirror 队列把数据同步到其他的 MQ 节点，这样来实现其高可靠

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

镜像模式的主要特点在于每个 mq 实例都包含一份完整的数据镜像，内部有一个 master 选举算法，通过 VIP 对外提供连接

*   consumer，任意连接一个节点，若连上的不是 master，请求会转发给 master，为了保证消息的可靠性，consumer 回复 ack 给 master 后，master 删除消息并广播所有的 slaver 去删除。
*   publisher ，任意连接一个节点，若连上的不是 master，则转发给 master，由 master 存储并转发给其他的 slaver 存储。 如果 master 挂掉，则从 slaver 中选择消息队列最长的为 master，

### 3.5 普通集群模式

exchange，buindling 再所有的节点上都会保存一份，但是 queue 只会存储在其中的一个节点上，但是所有的节点都会存储一份 queue 的 meta 信息

如果生产者连接的是另外一个节点，将会把消息转发到存储该队列的节点上。如果消费者连接了非存储队列的节点取数据，则从存储消息的节点拉取数据。

其核心特点在于：

*   数据拆分存储，若纯消息的节点挂了，则只能等待它恢复之后才能正常工作

### 3.6 多活模式

> 这个模式我的理解也不够深刻，以下内容来自于网上摘录，待后面到 rabbitmq 专题之后调研后进一步阐述

rabbitMQ 部署架构采用双中心模式 (多中心)，那么在两套(或多套) 数据中心各部署一套 rabbitMQ 集群，各中心的 rabbitMQ 服务除了需要为业务提供正常的消息服务外，中心之间还需要实现部分队列消息共享

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

federation 插件是一个不需要构建 cluster ，而在 brokers 之间传输消息的高性能插件，federation 插件可以在 brokers 或者 cluster 之间传输消息，连接的双方可以使用不同的 users 和 virtual hosts，双方也可以使用不同版本的 rabbitMQ 和 erlang。

federation 插件使用 AMQP 协议通信，可以接受不连续的传输。federation 不是建立在集群上的，而是建立在单个节点上的，如图上黄色的 rabbit node 3 可以与绿色的 node1、node2、node3 中的任意一个利用 federation 插件进行数据同步。

### 3.7 小结

rabbitmq 的高可用机制的方案也比较好理解

*   主备模式
*   镜像模式：全量冗余一份数据，主对外提供服务，可以实现自动切主
*   普通集群模式：数据拆分到集群的实例中，consumer/publisher 连接到实例之后，会从具体持有 exchange/topic 的实例上拉数据
*   远程模式：适用于多中心的场景，将消息转发给其他中心的实例

这里采用的高可用思路也无外乎常见的几种：持久化 + 数据冗余 + 拆分

相关博文：

*   [【MQ 系列】RabbitMq 核心知识点小结 | 一灰灰 Blog](https://link.juejin.cn?target=https%3A%2F%2Fspring.hhui.top%2Fspring-blog%2F2020%2F02%2F12%2F200212-SpringBoot%25E7%25B3%25BB%25E5%2588%2597%25E6%2595%2599%25E7%25A8%258B%25E4%25B9%258BRabbitMq%25E6%25A0%25B8%25E5%25BF%2583%25E7%259F%25A5%25E8%25AF%2586%25E7%2582%25B9%25E5%25B0%258F%25E7%25BB%2593%2F "https://spring.hhui.top/spring-blog/2020/02/12/200212-SpringBoot%E7%B3%BB%E5%88%97%E6%95%99%E7%A8%8B%E4%B9%8BRabbitMq%E6%A0%B8%E5%BF%83%E7%9F%A5%E8%AF%86%E7%82%B9%E5%B0%8F%E7%BB%93/")
*   [RabbitMQ 的 4 种集群架构](https://link.juejin.cn?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Fb7cc32b94d2a "https://www.jianshu.com/p/b7cc32b94d2a")
*   [rabbitmq 消息一致性问题 - Knight's Blog](https://link.juejin.cn?target=http%3A%2F%2Fwww.liaoqiqi.com%2Fpost%2F215 "http://www.liaoqiqi.com/post/215")

4. ElasticSearch 高可用方案
----------------------

接下来我们再看一下现在非常流行的分布式搜索引擎 ElasticSearch 是如何保证高可用的

### 4.1 集群

> Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎
> 
> by 官网描述

对于 es 而言，通常都是集群方式对外提供服务，每启动一个实例叫做一个节点 (Node)，每个节点会定义一个节点名 (Node Name)，集群名 (Cluster Name)，相同集群名的节点会构建为一个集群；

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

上图包含了 es 集群的核心要素：

*   每个节点包含集群名 + 节点名两个属性，相同集群名的节点挂在一个集群内
*   节点启动之后，开始 PING 其他节点（连接上后会得到对应节点所在集群的所有信息）
*   节点发现主要靠 Zen Discover 来实现，选举也是靠它来实现

选举主要流程如下

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

*   选举同样也是依赖 Zen Discover 来实现
*   每个节点上报自己任务的主节点，然后票数最多的就是主节点；票数相同的情况下，根据 ID 排序，选第一个

上面就是 es 集群的构建与主节点的选举过程；es 支持任意节点数目的集群（1- N），无法完全依赖投票的机制来选主，而是通过一个规则。

只要所有的节点都遵循同样的规则，得到的信息都是对等的，选出来的主节点肯定是一致的。

但分布式系统的问题就出在信息不对等的情况，这时候很容易出现脑裂（Split-Brain）的问题。

大多数解决方案就是设置一个 Quorum 值，要求可用节点必须大于 Quorum（一般是超过半数节点），才能对外提供服务。而 Elasticsearch 中，这个 Quorum 的配置就是 `discovery.zen.minimum_master_nodes`，当**候选主节点**的个数超过这个参数值时，开始选举，选主完成之后对外提供服务

ES 作为分布式、近实时搜索系统，天然支持集群的服务能力，通过 Zen Discover 来实现节点通信、集群管理、选主

### 4.2 脑裂问题

上面提到了脑裂，接下来简单看一下 ES 是如何解决脑裂问题的

> 脑裂：由于网络或者集群健康监测问题，导致整个集群出现多个 master 节点，这种现象就是脑裂

es 对节点进行了角色划分

*   数据节点：负责数据的存储和相关的操作 (CURD，聚合) 等，因此对机器性能要求较高
*   候选主节点：拥有选举权和被选举权，主节点在候选主节点中评选出来，负责创建索引、删除索引、跟踪哪些节点是群集的一部分，并决定哪些分片分配给哪些的节点、追踪集群中节点的状态等

> 一个节点，可以即是数据节点，又是候选主节点，但是注意它们两者的定位，主节点对机器性能要求没有数据节点高，当一台机器既是数据节点又是主节点时，可能出现长耗时、耗资源的请求导致主节点服务异常；
> 
> 通常更推荐的方案是使用性能低一点的作为候选主节点，性能高的作为数据节点

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

接下来看下脑裂出现的情况

*   网络问题，导致分区：即部分节点连接不到主节点，认为它挂了，然后选举出现的主节点
*   主节点负载、响应延迟：主节点由于负载过高、或者响应超时，导致重新选举新的主节点

解决方案：

*   适当调大 ping timeout 响应时间，避免因为网络、主节点性能问题导致的选举
*   设置最少选举节点数大于候选主节点的半数，这样只要有半数以上的候选节点存活，则可以选举出一个主节点；而当可用节点数小于半数时，不参与选举，集群无法使用，也不会出现状态异常的情况
*   角色分离：数据节点 + 候选主节点不放在一台机器上；

在有主节点的系统中，一般都需要考虑脑裂问题，常见的策略无非是：

*   半数节点以上的投票才算有效
*   es 额外提供了节点的角色定位，数据节点和候选主节点，其中只有候选主节点才有选举权和被选举权，提供一种角色分离的可选方案，来避免主节点被其他数据服务影响

### 4.3 数据分片

当数据量过大时，es 支持自动拆分，将一个索引的上数据水平拆分到不同的数据块 -- 分片 (Shards)，为了提供可用性，每个索引在定义时除了分片之外，还会定义副本数量，这里的副本可以理解为数据冗余，其中副本和分片必然不在一个节点上，在主节点异常时，副本可以提供数据查询能力

> es 默认在创建索引时，分片数为 5，每个分片对应一个副本

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

ES 通过分片，将索引数据水平拆分，分片数越多，每个分片上的数据量就越少；而副本则是对应的每个分片的冗余，可以理解为主备，副本越多，消耗则越大

两点小说明

*   对应副本的概念，上面的分片也叫做主分片
*   当一个数据写入 / 更新到分片时，只有所有的副本都更新完毕之后，才算完成（可以 MySql 的全同步）

### 4.4 数据持久化

最后再说一下 es 的持久化机制，与前面先说持久化不同，es 这里则需要先了解上面的基本流程，索引数据需要保存到主分片上，最终落盘，接下来看一下完整的流程

**主分片数据更新流程**

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

简述一下上面的流程

*   首先请求随机连一个 es 节点（这个节点叫做协调节点），然后通过路由算法，确定数据对应的主分片
*   写数据到主分片，然后同步到副本（多个副本时采用并发同步，乐观锁控制）
*   所有副本同步完成之后，主分片节点告诉协调节点最终结果，然后协调节点告诉调用者响应

当数据写入到主分片上之后，接下来再看一下这个数据时如何刷新到磁盘上的

**分段存储**

索引文档以段的形式存储磁盘，即一个索引文件会划分为很多个子文件，这里的子文件就是段

> 每一个段本身都是一个倒排索引，并且段具有不变性，一旦索引的数据被写入硬盘，就不可再修改；段被写入到磁盘后会生成一个提交点，提交点是一个用来记录所有提交后段信息的文件

段的特性，有下面几个有点

*   分段存储，可以有效避免读写时加锁的问题
*   不变性，数据只读可以高效缓存，无需考虑更新
*   一个段一旦拥有了提交点，就说明这个段只有读的权限，失去了写的权限。相反，当段在内存中时，就只有写的权限，而不具备读数据的权限，意味着不能被检索

由于段不可变，所以在更新时需要额外处理

*   新增：当前文档新增一个段
*   删除：新增一个. del 文件，记录被删除的文档信息；被标记删除的文档仍然可以被检索到，只是最终返回时被移除
*   更新：删除文件中标记旧的文档删除，插入新的段

**延迟写**

ES 并不会实时将内存中的数据写入段，而是采用延迟写的策略（类似前面的写 buffer，然后异步定时刷盘）

es 先将内存数据，写入文件缓存系统 (操作系统内存)，

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

> 上图来自 * [两万字教程，带你遨游 ElasticSearch](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FgvSNazpxAE78v0J7DP9K1g "https://mp.weixin.qq.com/s/gvSNazpxAE78v0J7DP9K1g")

注意几个事项

*   写入文件缓存系统，之后异步落盘，可能导致丢数据，es 采用事务日志的方式来处理恢复策略 (即 mysql 的先写日志，崩溃之后做回放恢复)
*   es 对外服务时，检索文件缓存系统 + 段中的文档，而内存中的数据不会被检索到（所以所 es 是近实时搜索引擎，因为最新写入的数据还在内存中，没有提交，立马查就查不到）
*   为了避免段过多，es 会定时做合并，将很多小的段合并成大的段（合并过程中会自动移除被标记删除的文档）

最后小结一下 es 的持久化

*   索引分段存储，段生成 checkpoint 之后，则只读，因此可以全量缓存，不用考虑更新修改
*   延迟写策略：先更新内存数据，异步提交文件缓存系统，最后再由操作系统刷盘
*   内存中的数据不能被检索；文件缓存 + 段中的数据提供查询聚合，最终的结果会过滤已标记删除的文档

### 4.5 小结

这一小节主要介绍的是 ES 的高可用机制，包括 ES 的集群工作原理，选举策略；采用数据分片支持大数据场景的支持，借助副本来提高可用性；

ES 原生支持集群

*   角色划分：候选主节点 + 数据节点
*   数据节点：负责数据的存储和相关的操作 (CURD，聚合) 等，因此对机器性能要求较高
*   候选主节点：拥有选举权和被选举权，主节点在候选主节点中评选出来，负责创建索引、删除索引、跟踪哪些节点是群集的一部分，并决定哪些分片分配给哪些的节点、追踪集群中节点的状态等

ES 数据持久化策略

*   索引分段存储，段生成 checkpoint 之后，则只读，因此可以全量缓存，不用考虑更新修改；当出现修改时，标记原来段中文档删除，在新的段写入数据
*   延迟写策略：先更新内存数据，异步提交文件缓存系统，最后再由操作系统刷盘
*   内存中的数据不能被检索；文件缓存 + 段中的数据提供查询聚合，最终的结果会过滤已标记删除的文档

**参考博文**

*   [两万字教程，带你遨游 ElasticSearch](https://link.juejin.cn?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FgvSNazpxAE78v0J7DP9K1g "https://mp.weixin.qq.com/s/gvSNazpxAE78v0J7DP9K1g")
*   [ElasticSearch 原理解析_elasticsearch_Chank](https://link.juejin.cn?target=https%3A%2F%2Fxie.infoq.cn%2Farticle%2F55095e9626718380c4072f5fb "https://xie.infoq.cn/article/55095e9626718380c4072f5fb")

5. 一灰灰的总结
---------

### 5.1 综述

本片文章主要是分析当下不同应用场景下的几个主流系统的高可用策略，来看一下如何来保障的系统的高可用

**常见的高可用思路**

*   冗余 （如数据副本、主备服务等）
*   拆分 （数据拆分、服务能力拆分等）
*   持久化

**redis**

*   持久化：RDB 数据落盘加载方式 + AOF 记录操作命令用于回放策略
*   主从，主从从：全量数据冗余、读写请求分离，负载均衡的思想；核心问题在于主节点挂掉之后需要人工参与手动指定主库
*   哨兵机制：PING/PONG 的探活机制，监听主节点，宕机之后自动选主，确保高可用；核心问题在于所有的实例冗余相同的一份数据，数据量大时不友好
*   集群：数据分片，每个实例提供部分服务能力

**mysql**

*   通过冗余来实现高可用：如主备
*   读写分离，实现负载均衡：主从、主从从模式
*   数据持久化策略：操作内存 (buffer)，异步刷盘，两阶段提交保障一致性

**rabbitmq**

*   主备模式
*   镜像模式：全量冗余一份数据，主对外提供服务，可以实现自动切主
*   普通集群模式：数据拆分到集群的实例中，consumer/publisher 连接到实例之后，会从具体持有 exchange/topic 的实例上拉数据
*   远程模式：适用于多中心的场景，将消息转发给其他中心的实例

**ElasticSearch**

*   ES 集群：数据节点 + 候选主节点
*   ES 持久化：
    *   延迟写策略，先更新内存，然后提交操作系统缓存，最后异步刷新到磁盘；
    *   索引分段存储：段生成 checkpoint 之后，则只读，因此可以全量缓存，不用考虑更新修改；当出现修改时，标记原来段中文档删除，在新的段写入数据

### 5.2 主题无关

在准备写本文时，原计划针对不同业务场景各挑一个经典的系统来分析下各自的高可用方案，实际写下来发现工作量有点大；就把最后的一个分布式文件系统 hdfs 给暂缓了（对于大多数业务开发而言，接触的机会也不会太多），这个会放在《分布式系统 - 案例剖析》中进行介绍

最近会花大量的时间精力，准备做一个高质量的《分布式专栏》，欢迎有兴趣收藏关注 一灰灰的主站

*   专栏地址：* [分布式专栏 | https://hhui.top / 分布式](https://link.juejin.cn?target=https%3A%2F%2Fhhui.top%2F%25E5%2588%2586%25E5%25B8%2583%25E5%25BC%258F%2F "https://hhui.top/%E5%88%86%E5%B8%83%E5%BC%8F/")
*   精选： * [分布式设计模式综述 | 一灰灰 Learning](https://link.juejin.cn?target=https%3A%2F%2Fhhui.top%2F%25E5%2588%2586%25E5%25B8%2583%25E5%25BC%258F%2F%25E8%25AE%25BE%25E8%25AE%25A1%25E6%25A8%25A1%25E5%25BC%258F%2F01.%25E5%2588%2586%25E5%25B8%2583%25E5%25BC%258F%25E8%25AE%25BE%25E8%25AE%25A1%25E6%25A8%25A1%25E5%25BC%258F%25E7%25BB%25BC%25E8%25BF%25B0%2F "https://hhui.top/%E5%88%86%E5%B8%83%E5%BC%8F/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/01.%E5%88%86%E5%B8%83%E5%BC%8F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E7%BB%BC%E8%BF%B0/")

* 本文所有引用博文地址，请点击原文查看

> 最后插个小广告，感兴趣的小伙伴可以我的公众号：“一灰灰 blog”