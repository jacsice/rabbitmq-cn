####内容摘要

## [Overview](https://www.rabbitmq.com/clustering.html#overview)

>This guide covers fundamental topics related to RabbitMQ clustering:
>
>- How RabbitMQ nodes are identified: [node names](https://www.rabbitmq.com/clustering.html#node-names)
>- [Requirements](https://www.rabbitmq.com/clustering.html#cluster-formation-requirements) for clustering
>- What data is and isn't [replicated between cluster nodes](https://www.rabbitmq.com/clustering.html#cluster-membership)
>- What clustering [means for clients](https://www.rabbitmq.com/clustering.html#clustering-and-clients)
>- [How clusters are formed](https://www.rabbitmq.com/clustering.html#cluster-formation)
>- How nodes [authenticate to each other](https://www.rabbitmq.com/clustering.html#erlang-cookie) (and with CLI tools)
>- Why [two cluster nodes are highly recommended against](https://www.rabbitmq.com/clustering.html#node-count)
>- [Node restarts](https://www.rabbitmq.com/clustering.html#restarting) and how nodes rejoin their cluster
>- How to [remove a cluster node](https://www.rabbitmq.com/clustering.html#removing-nodes)
>- How to [reset a cluster node](https://www.rabbitmq.com/clustering.html#resetting-nodes)
>
>and more. [Cluster Formation and Peer Discovery](https://www.rabbitmq.com/cluster-formation.html) is a closely related guide that focuses on peer discovery and cluster formation automation-related topics. For queue contents (message) replication, see the [Mirrored Queues](https://www.rabbitmq.com/ha.html) guide.
>
>A RabbitMQ cluster is a logical grouping of one or several nodes, each sharing users, virtual hosts, queues, exchanges, bindings, runtime parameters and other distributed state.

本指南涵盖了与RabbitMQ集群相关的内容:

* 如何识别RabbitMQ节点：节点名字
* 集群配置
* 集群节点之前什么数据不会被同步
* 集群对客户端来说意味着什么
* 集群的组成
* 节点之前如何相互认证
* 为什么强烈建议不要使用两个集群节点
* 节点重启，节点如何重新加入集群
* 如何删除一个集群节点
* 如何重启一个集群节点
* 等等

RabbitMQ集群是一个或多个节点的逻辑分组，每个节点共享用户、虚拟主机、队列、exchanges、bindings、运行时参数和其他分布式状态。



####集群组件

>## [Cluster Formation](https://www.rabbitmq.com/clustering.html#cluster-formation)
>
>### [Ways of Forming a Cluster](https://www.rabbitmq.com/clustering.html#cluster-formation-options)
>
>A RabbitMQ cluster can formed in a number of ways:
>
>- Declaratively by listing cluster nodes in [config file](https://www.rabbitmq.com/configure.html)
>- Declaratively using DNS-based discovery
>- Declaratively using [AWS (EC2) instance discovery](https://github.com/rabbitmq/rabbitmq-peer-discovery-aws) (via a plugin)
>- Declaratively using [Kubernetes discovery](https://github.com/rabbitmq/rabbitmq-peer-discovery-k8s) (via a plugin)
>- Declaratively using [Consul-based discovery](https://github.com/rabbitmq/rabbitmq-peer-discovery-consul) (via a plugin)
>- Declaratively using [etcd-based discovery](https://github.com/rabbitmq/rabbitmq-peer-discovery-etcd) (via a plugin)
>- Manually with rabbitmqctl
>
>Please refer to the [Cluster Formation guide](https://www.rabbitmq.com/cluster-formation.html) for details.
>
>The composition of a cluster can be altered dynamically. All RabbitMQ brokers start out as running on a single node. These nodes can be joined into clusters, and subsequently turned back into individual brokers again.

组建集群的方式

* 在配置文件中列出所有的节点
* DNS-based, AWS, k8s, consul-based, etcd-based 
* rabbitmqctl

更多详细信息请看[Cluster Formation guide](https://www.rabbitmq.com/cluster-formation.html) 

集群的组成可以动态的改变。所有RabbitMQ broker一开始都运行在单个节点上。可以将这些节点加入到集群中，然后还是独立的broker。



####节点名字

>### [Node Names (Identifiers)](https://www.rabbitmq.com/clustering.html#node-names)
>
>RabbitMQ nodes are identified by node names. A node name consists of two parts, a prefix (usually rabbit) and hostname. For example,` rabbit@node1.messaging.svc.local` is a node name with the prefix of rabbit and hostname of node1.messaging.svc.local.
>
>Node names in a cluster must be unique. If more than one node is running on a given host (this is usually the case in development and QA environments), they must use different prefixes, e.g. rabbit1@hostname and rabbit2@hostname.
>
>In a cluster, nodes identify and contact each other using node names. This means that the hostname part of every node name [must resolve](https://www.rabbitmq.com/clustering.html#hostname-resolution-requirement). [CLI tools](https://www.rabbitmq.com/cli.html) also identify and address nodes using node names.
>
>When a node starts up, it checks whether it has been assigned a node name. This is done via the RABBITMQ_NODENAME [environment variable](https://www.rabbitmq.com/configure.html#supported-environment-variables). If no value was explicitly configured, the node resolves its hostname and prepends rabbit to it to compute its node name.
>
>If a system uses fully qualified domain names (FQDNs) for hostnames, RabbitMQ nodes and CLI tools must be configured to use so called long node names. For server nodes this is done by setting the RABBITMQ_USE_LONGNAME [environment variable](https://www.rabbitmq.com/configure.html#supported-environment-variables) to true.
>
>For CLI tools, either RABBITMQ_USE_LONGNAME must be set or the --longnames option must be specified.

RabbitMQ节点通过节点名字来标示，一个节点名字包含两部分，前缀(一般是rabbit)和主机名。例如，`rabbit@node1.messaging.svc.local`是主机名为`node1.messaging.svc.local`的节点名字。

集群中的节点名字必须是唯一的，如果一个主机上运行了多个节点(比如开发和QA环境)，他们必须使用不同的前缀，例如`rabbit1@hostname` 和 `rabbit2@hostname`.

在集群中，节点之前通过节点名字来识别和交流，这以为着节点名的主机名部分必须可以被解析。CLI 工具也通过节点名字来识别和定位节点。

当一个节点启动的时候，它会先检查是否指定了一个节点名字，通过RABBITMQ_NODENAME这个环境变量来指定，如果这个值没有被明确的配置，节点会分析它的主机名，并加上rabbit来组成它的节点名。

如果系统使用完全限定域名(FQDNs)作为主机名，则RabbitMQ节点和CLI工具必须配置RABBITMQ_USE_LONGNAME为true

对于CLI工具，RABBITMQ_USE_LONGNAME必须被设置或者--longnames要被指定



> ## [Cluster Formation Requirements](https://www.rabbitmq.com/clustering.html#cluster-formation-requirements)
>
> ### [Hostname Resolution](https://www.rabbitmq.com/clustering.html#hostname-resolution-requirement)
>
> RabbitMQ nodes address each other using domain names, either short or fully-qualified (FQDNs). Therefore hostnames of all cluster members must be resolvable from all cluster nodes, as well as machines on which command line tools such as rabbitmqctl might be used.
>
> Hostname resolution can use any of the standard OS-provided methods:
>
> - DNS records
> - Local host files (e.g. /etc/hosts)
>
> In more restrictive environments, where DNS record or hosts file modification is restricted, impossible or undesired, [Erlang VM can be configured to use alternative hostname resolution methods](http://erlang.org/doc/apps/erts/inet_cfg.html), such as an alternative DNS server, a local file, a non-standard hosts file location, or a mix of methods. Those methods can work in concert with the standard OS hostname resolution methods.
>
> To use FQDNs, see RABBITMQ_USE_LONGNAME in the [Configuration guide](https://www.rabbitmq.com/configure.html#supported-environment-variables). See [Node Names](https://www.rabbitmq.com/clustering.html#node-names) above.

集群配置

主机名解析

RabbitMQ节点之间通过域名来定位彼此，短连接或者FQDNs都可以，因此，所有群集成员的主机名以及使用命令行工具（例如rabbitmqctl）的机器都必须是可解析的。

可以用系统提供的方法来解析主机名

* DNS记录
* 本地的主机文件 /etc/hosts

在更具限制性的环境中，DNS记录或主机文件的修改都是受到限制的，不能存在的，不希望有的

pass



#### 端口访问

>## [Port Access](https://www.rabbitmq.com/clustering.html#ports)
>
>RabbitMQ nodes bind to ports (open server TCP sockets) in order to accept client and CLI tool connections. Other processes and tools such as SELinux may prevent RabbitMQ from binding to a port. When that happens, the node will fail to start.
>
>CLI tools, client libraries and RabbitMQ nodes also open connections (client TCP sockets). Firewalls can prevent nodes and CLI tools from communicating with each other. Make sure the following ports are accessible:
>
>- 4369: [epmd](https://www.rabbitmq.com/networking.html#epmd), a helper discovery daemon used by RabbitMQ nodes and CLI tools
>- 5672, 5671: used by AMQP 0-9-1 and 1.0 clients without and with TLS
>- 25672: used for inter-node and CLI tools communication (Erlang distribution server port) and is allocated from a dynamic range (limited to a single port by default, computed as AMQP port + 20000). Unless external connections on these ports are really necessary (e.g. the cluster uses [federation](https://www.rabbitmq.com/federation.html) or CLI tools are used on machines outside the subnet), these ports should not be publicly exposed. See [networking guide](https://www.rabbitmq.com/networking.html) for details.
>- 35672-35682: used by CLI tools (Erlang distribution client ports) for communication with nodes and is allocated from a dynamic range (computed as server distribution port + 10000 through server distribution port + 10010). See [networking guide](https://www.rabbitmq.com/networking.html) for details.
>- 15672: [HTTP API](https://www.rabbitmq.com/management.html) clients, [management UI](https://www.rabbitmq.com/management.html) and [rabbitmqadmin](https://www.rabbitmq.com/management-cli.html) (only if the [management plugin](https://www.rabbitmq.com/management.html) is enabled)
>- 61613, 61614: [STOMP clients](https://stomp.github.io/stomp-specification-1.2.html) without and with TLS (only if the [STOMP plugin](https://www.rabbitmq.com/stomp.html) is enabled)
>- 1883, 8883: ([MQTT clients](http://mqtt.org/) without and with TLS, if the [MQTT plugin](https://www.rabbitmq.com/mqtt.html) is enabled
>- 15674: STOMP-over-WebSockets clients (only if the [Web STOMP plugin](https://www.rabbitmq.com/web-stomp.html) is enabled)
>- 15675: MQTT-over-WebSockets clients (only if the [Web MQTT plugin](https://www.rabbitmq.com/web-mqtt.html) is enabled)
>- 15692: Prometheus metrics (only if the [Prometheus plugin](https://www.rabbitmq.com/prometheus.html) is enabled)
>
>It is possible to [configure RabbitMQ](https://www.rabbitmq.com/configure.html) to use [different ports and specific network interfaces](https://www.rabbitmq.com/networking.html).

RabbitMQ节点绑定接口(打开服务器TCP套接字)是为了能接收client和CLI工具的连接，其他的进程或者工具，就想SELinux可能会阻止RabbitMQ绑定到端口，当这些发生时，节点将启动失败。

CLI工具，客户端库和RabbitMQ节点会打开连接(客户端TCP套接字)，防火墙会阻止节点和CLI工具彼此之间的通信，确保以下的端口是可访问的。

* 4369 epmd, RabbitMQ节点和CLI工具使用的辅助发现守护进程
* 5672, 5671  AMQP 0-9-1 and 1.0客户端不使用SSL和使用SSL时的端口
* 15672 HTTP接口客户端，管理UI，rabbitmqadmin(必须安装management plugin)

* 略



####集群中的节点

>## [Nodes in a Cluster](https://www.rabbitmq.com/clustering.html#cluster-membership)
>
>### [What is Replicated?](https://www.rabbitmq.com/clustering.html#overview-what-is-replicated)
>
>All data/state required for the operation of a RabbitMQ broker is replicated across all nodes. An exception to this are message queues, which by default reside on one node, though they are visible and reachable from all nodes. To replicate queues across nodes in a cluster, see the documentation on [high availability](https://www.rabbitmq.com/ha.html) (note: this guide is a prerequisite for mirroring).
>
>### [Nodes are Equal Peers](https://www.rabbitmq.com/clustering.html#peer-equality)
>
>Some distributed systems have leader and follower nodes. This is generally not true for RabbitMQ. All nodes in a RabbitMQ cluster are equal peers: there are no special nodes in RabbitMQ core. This topic becomes more nuanced when [queue mirroring](https://www.rabbitmq.com/ha.html) and plugins are taken into consideration but for most intents and purposes, all cluster nodes should be considered equal
>
>Many [CLI tool](https://www.rabbitmq.com/cli.html) operations can be executed against any node. An [HTTP API](https://www.rabbitmq.com/management.html) client can target any cluster node.
>
>Individual plugins can designate (elect) certain nodes to be "special" for a period of time. For example, [federation links](https://www.rabbitmq.com/federation.html) are colocated on a particular cluster node. Should that node fail, the links will be restarted on a different node.
>
>In versions older than 3.6.7, [RabbitMQ management plugin](https://www.rabbitmq.com/management.html) used a dedicated node for stats collection and aggregation.

#####什么是可复制的

RabbitMQ broker操作所需的所有数据/状态可以被复制到所有节点。消息队列是一个例外，它默认驻留在一个节点上，但是它们是可见的，并且可以从所有节点到达。跨集群的节点队列复制，请看文档[high_availability](high_availability.md) (note: 这个文档是mirroring的基础)。

#####节点是平等的

有些分布式系统有主从节点，RabbitMQ并不是这样的。RabbitMQ集群中的所有节点都是平等的，没有特殊的节点。当镜像队列和插件被考虑在内时，这个主题变得更加微妙，但是对于大多数意图和目的，应该认为所有集群节点都是平等的。

可以针对任何节点执行许多CLI工具操作，HTTP接口客户端可以请求任意的集群节点。

独立的插件可以在一段时间内指定(选择)某些节点为“特殊”节点。例如，federation links 在一个特定的集群节点上进行协作。如果该节点失败，将在另一个节点上重新启动链接。?



####节点之间如何相互认证：Erlang Cookie

> ### [How CLI Tools Authenticate to Nodes (and Nodes to Each Other): the Erlang Cookie](https://www.rabbitmq.com/clustering.html#erlang-cookie)
>
> RabbitMQ nodes and CLI tools (e.g. rabbitmqctl) use a cookie to determine whether they are allowed to communicate with each other. For two nodes to be able to communicate they must have the same shared secret called the Erlang cookie. The cookie is just a string of alphanumeric characters up to 255 characters in size. It is usually stored in a local file. The file must be only accessible to the owner (e.g. have UNIX permissions of 600 or similar). Every cluster node must have the same cookie.
>
> If the file does not exist, Erlang VM will try to create one with a randomly generated value when the RabbitMQ server starts up. Using such generated cookie files are appropriate in development environments only. Since each node will generate its own value independently, this strategy is not really viable in a [clustered environment](https://www.rabbitmq.com/clustering.html).
>
> Erlang cookie generation should be done at cluster deployment stage, ideally using automation and orchestration tools such as Chef, Puppet, BOSH, Docker or similar.

RabbitMQ节点和CLI工具(例如：rabbitmqctl) 使用cookie来决定他们是否被允许与彼此通信。两个节点之间如果要通信，他们必须有一个叫做的Erlang cookie的相同的共享秘钥，Cookie只是一串字母数字字符，最大长度为255个字符，它一般存储在本地文件中，这个文件只能被所有者访问(unix权限未600，即rw-)，集群中的每个节点必须有相同的cookie。

如果该文件不存在，则在RabbitMQ服务器启动时，Erlang VM将尝试使用随机生成的值创建一个文件。仅在开发环境中使用此类生成的cookie文件是合适的。由于每个节点将独立生成自己的值，这个策略在集群环境中不可用。

Erlang cookie的生成应在集群部署阶段完成，理想情况下使用自动化和编排工具，例如Chef，Puppet，BOSH，Docker或类似工具。



####cookie文件的位置

>### [Cookie File Locations](https://www.rabbitmq.com/clustering.html#cookie-file-locations)
>
>#### Linux, MacOS, *BSD
>
>On UNIX systems, the cookie will be typically located in /var/lib/rabbitmq/.erlang.cookie (used by the server) and $HOME/.erlang.cookie (used by CLI tools)

在unix系统上，文件一般在`/var/lib/rabbitmq/.erlang.cookie` (服务器使用) 和 `$HOME/.erlang.cookie`（CLI工具使用）



####认证失败

### [Authentication Failures](https://www.rabbitmq.com/clustering.html#cli-authentication-failures)

> When the cookie is misconfigured (for example, not identical), RabbitMQ will log errors such as "Connection attempt from disallowed node" and "Could not auto-cluster". When a [CLI tool](https://www.rabbitmq.com/cli.html) such as rabbitmqctl fails to authenticate with RabbitMQ, the message usually says



```ini
* epmd reports node 'rabbit' running on port 25672
* TCP connection succeeded but Erlang distribution failed
* suggestion: hostname mismatch?
* suggestion: is the cookie set correctly?
* suggestion: is the Erlang distribution using TLS?
```



An incorrectly placed cookie file or cookie value mismatch are most common scenarios for such failures.

When a recent Erlang/OTP version is used, authentication failures contain more information and cookie mismatches can be identified better:

```ini
* connected to epmd (port 4369) on warp10
* epmd reports node 'rabbit' running on port 25672
* TCP connection succeeded but Erlang distribution failed

* Authentication failed (rejected by the remote node), please check the Erlang cookie
```

See the [CLI Tools guide](https://www.rabbitmq.com/cli.html) for more information.

