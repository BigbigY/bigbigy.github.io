---
title: 初识Kubernetes
tags: [Go]
categories: 基于k8s的私有容器云从0到1的建设之路
type: "categories"
author: "luck"

---


# 什么是Kubernetes？
> Kubernetes（简称k8s，命名由来是因为k与s中间有8个字母）是Google基于内部Borg开源的容器编排引擎，它支持自动化部署、大规模可伸缩、应用容器化管理。我们在完成一个应用程序的开发时，需要冗余部署该应用的多个实例，同时需要支持对应用的请求进行负载均衡，在Kubernetes中，我们可以把这个应用的多个实例分别启动一个容器，每个容器里面运行一个应用实例，然后通过内置的负载均衡策略，实现对这一组应用实例的管理、发现、访问，而这些细节都不需要应用开发和运维人员去进行复杂的手工配置和处理。


<div><video class="wp-video-shortcode" id="video-227-1" width="640" height="360" preload="metadata" controls="controls"><source type="video/mp4" src="https://dn-linuxcn.qbox.me/The%20Illustrated%20Children%27s%20Guide%20to%20Kubernetes-4ht22ReBjno.mp4?_=1" /><a href="https://dn-linuxcn.qbox.me/The%20Illustrated%20Children%27s%20Guide%20to%20Kubernetes-4ht22ReBjno.mp4">https://dn-linuxcn.qbox.me/The%20Illustrated%20Children%27s%20Guide%20to%20Kubernetes-4ht22ReBjno.mp4</a></video></div>

# 基本概念

#### Pod
Pod是Kubernetes集群中所有业务类型的基础，可以看作运行在k8s集群中的小机器人，不同类型的业务就需要不同类型的小机器人去执行。

#### 副本控制器（Replication Controller，RC）
RC是Kubernetes集群中最早的保证Pod高可用的API对象。通过监控运行中的Pod来保证集群中运行指定数目的Pod副本。指定的数目可以是多个也可以是1个；少于指定数目，RC就会启动运行新的Pod副本；多于指定数目，RC则会杀掉多于的Pod副本。通过RC运行Pod也比直接运行Pod更明智，因为RC也可以发挥它高可用的能力，保证永远有1个Pod在运行。RC是Kubernetes较早期的技术概念，只适用于长期伺服型的业务类型。

#### 副本集（Replica Set，RS）
RS是新一代RC，提供同样的高可用能力，区别主要在于RS后来居上，能支持更多种类型的匹配模式。副本集对象一般不单独使用，而是作为Deployment的理想状态参数使用。

#### 部署（Deployment）
部署表示用户对Kubernetes集群的一次更新操作。部署是一个比RS应用模式更广的API对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。滚动升级一个服务，实际是创建一个新的RS，然后逐渐将新RS中副本数增加到理想状态，将旧RS中副本数减少到0的复合状态；这样一个复合操作用一个RS是不太好描述的，所以用一个更通用的Deployment来描述。以Kubernetes的发展方向，未来对所有长期伺服型的的业务的管理，都会通过Deployment来管理。

#### 服务（Service）
RC、RS和Deployment只是保证了支撑服务的微服务Pod的数量，但是没有解决如何访问这些服务的问题。一个Pod只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的IP启动一个新的Pod，因此不能以确定的IP和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。在K8集群中，客户端需要访问的服务就是Service对象。每个Service会对应一个集群内部有效的虚拟IP，集群内部通过虚拟IP访问一个服务。在Kubernetes集群中微服务的负载均衡是由Kube-proxy实现的。Kube-proxy是Kubernetes集群内部的负载均衡器。它是一个分布式代理服务器，在Kubernetes的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的Kube-proxy就越多，高可用节点也随之增多。与之相比，我们平时在服务器端做个反向代理做负载均衡，还要进一步解决反向代理的负载均衡和高可用问题。

#### 任务（Job）
Job是Kubernetes用来控制批处理型任务的API对象。批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。Job管理的Pod根据用户的设置把任务成功完成就自动退出了。成功完成的标志根据不同的spec.completions策略而不同：单Pod型任务有一个Pod成功就标志完成；定数成功型任务保证有N个任务全部成功；工作队列型任务根据应用确认的全局成功而标志成功。

#### 后台支撑服务集（DaemonSet）
长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的Pod，有些节点上又没有这类Pod运行；而后台支撑型服务的核心关注点在Kubernetes集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类Pod运行。节点可能是所有集群节点也可能是通过nodeSelector选定的一些特定节点。典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持Kubernetes集群运行的服务。


#### 有状态服务集（PetSet）
Kubernetes在1.3版本里发布了Alpha版的PetSet功能。在云原生应用的体系里，有下面两组近义词；第一组是无状态（stateless）、牲畜（cattle）、无名（nameless）、可丢弃（disposable）；第二组是有状态（stateful）、宠物（pet）、有名（having name）、不可丢弃（non-disposable）。RC和RS主要是控制提供无状态服务的，其所控制的Pod的名字是随机设置的，一个Pod出故障了就被丢弃掉，在另一个地方重启一个新的Pod，名字变了、名字和启动在哪儿都不重要，重要的只是Pod总数；而PetSet是用来控制有状态服务，PetSet中的每个Pod的名字都是事先确定的，不能更改。PetSet中Pod的名字的作用，并不是《千与千寻》的人性原因，而是关联与该Pod对应的状态。
对于RC和RS中的Pod，一般不挂载存储或者挂载共享存储，保存的是所有Pod共享的状态，Pod像牲畜一样没有分别（这似乎也确实意味着失去了人性特征）；对于PetSet中的Pod，每个Pod挂载自己独立的存储，如果一个Pod出现故障，从其他节点启动一个同样名字的Pod，要挂在上原来Pod的存储继续以它的状态提供服务。
适合于PetSet的业务包括数据库服务MySQL和PostgreSQL，集群化管理服务Zookeeper、etcd等有状态服务。PetSet的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制。传统的虚拟机正是一种有状态的宠物，运维人员需要不断地维护它，容器刚开始流行时，我们用容器来模拟虚拟机使用，所有状态都保存在容器里，而这已被证明是非常不安全、不可靠的。使用PetSet，Pod仍然可以通过漂移到不同节点提供高可用，而存储也可以通过外挂的存储来提供高可靠性，PetSet做的只是将确定的Pod与确定的存储关联起来保证状态的连续性。PetSet还只在Alpha阶段，后面的设计如何演变，我们还要继续观察。

#### 集群联邦（Federation）
Kubernetes在1.3版本里发布了beta版的Federation功能。在云计算环境中，服务的作用距离范围从近到远一般可以有：同主机（Host，Node）、跨主机同可用区（Available Zone）、跨可用区同地区（Region）、跨地区同服务商（Cloud Service Provider）、跨云平台。Kubernetes的设计定位是单一集群在同一个地域内，因为同一个地区的网络性能才能满足Kubernetes的调度和计算存储连接要求。而联合集群服务就是为提供跨Region跨服务商Kubernetes集群服务而设计的。
每个Kubernetes Federation有自己的分布式存储、API Server和Controller Manager。用户可以通过Federation的API Server注册该Federation的成员Kubernetes Cluster。当用户通过Federation的API Server创建、更改API对象时，Federation API Server会在自己所有注册的子Kubernetes Cluster都创建一份对应的API对象。在提供业务请求服务时，Kubernetes Federation会先在自己的各个子Cluster之间做负载均衡，而对于发送到某个具体Kubernetes Cluster的业务请求，会依照这个Kubernetes Cluster独立提供服务时一样的调度模式去做Kubernetes Cluster内部的负载均衡。而Cluster之间的负载均衡是通过域名服务的负载均衡来实现的。
所有的设计都尽量不影响Kubernetes Cluster现有的工作机制，这样对于每个子Kubernetes集群来说，并不需要更外层的有一个Kubernetes Federation，也就是意味着所有现有的Kubernetes代码和机制不需要因为Federation功能有任何变化。

#### 存储卷（Volume）
Kubernetes集群中的存储卷跟Docker的存储卷有些类似，只不过Docker的存储卷作用范围为一个容器，而Kubernetes的存储卷的生命周期和作用范围是一个Pod。每个Pod中声明的存储卷由Pod中的所有容器共享。Kubernetes支持非常多的存储卷类型，特别的，支持多种公有云平台的存储，包括AWS，Google和Azure云；支持多种分布式存储包括GlusterFS和Ceph；也支持较容易使用的主机本地目录hostPath和NFS。Kubernetes还支持使用Persistent Volume Claim即PVC这种逻辑存储，使用这种存储，使得存储的使用者可以忽略后台的实际存储技术（例如AWS，Google或GlusterFS和Ceph），而将有关存储实际技术的配置交给存储管理员通过Persistent Volume来配置。

#### 持久存储卷（Persistent Volume，PV）和持久存储卷声明（Persistent Volume Claim，PVC）
PV和PVC使得Kubernetes集群具备了存储的逻辑抽象能力，使得在配置Pod的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给PV的配置者，即集群的管理者。存储的PV和PVC的这种关系，跟计算的Node和Pod的关系是非常类似的；PV和Node是资源的提供者，根据集群的基础设施变化而变化，由Kubernetes集群管理员配置；而PVC和Pod是资源的使用者，根据业务服务的需求变化而变化，有Kubernetes集群的使用者即服务的管理员来配置。

#### 节点（Node）
Kubernetes集群中的计算能力由Node提供，最初Node称为服务节点Minion，后来改名为Node。Kubernetes集群中的Node也就等同于Mesos集群中的Slave节点，是所有Pod运行所在的工作主机，可以是物理机也可以是虚拟机。不论是物理机还是虚拟机，工作主机的统一特征是上面要运行kubelet管理节点上运行的容器。

#### 密钥对象（Secret）
Secret是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。使用Secret的好处是可以避免把敏感信息明文写在配置文件里。在Kubernetes集群中配置和使用服务不可避免的要用到各种敏感信息实现登录、认证等功能，例如访问AWS存储的用户名密码。为了避免将类似的敏感信息明文写在所有需要使用的配置文件中，可以将这些信息存入一个Secret对象，而在配置文件中通过Secret对象引用这些敏感信息。这种方式的好处包括：意图明确，避免重复，减少暴漏机会。

#### 用户帐户（User Account）和服务帐户（Service Account）
顾名思义，用户帐户为人提供账户标识，而服务账户为计算机进程和Kubernetes集群中运行的Pod提供账户标识。用户帐户和服务帐户的一个区别是作用范围；用户帐户对应的是人的身份，人的身份与服务的namespace无关，所以用户账户是跨namespace的；而服务帐户对应的是一个运行中程序的身份，与特定namespace是相关的。

#### 命名空间（Namespace）
命名空间为Kubernetes集群提供虚拟的隔离作用，Kubernetes集群初始有两个命名空间，分别是默认命名空间default和系统命名空间kube-system，除此以外，管理员可以可以创建新的命名空间满足需要。

#### RBAC访问授权
Kubernetes在1.3版本中发布了alpha版的基于角色的访问控制（Role-based Access Control，RBAC）的授权模式。相对于基于属性的访问控制（Attribute-based Access Control，ABAC），RBAC主要是引入了角色（Role）和角色绑定（RoleBinding）的抽象概念。在ABAC中，Kubernetes集群中的访问策略只能跟用户直接关联；而在RBAC中，访问策略可以跟某个角色关联，具体的用户在跟一个或多个角色相关联。显然，RBAC像其他新功能一样，每次引入新功能，都会引入新的API对象，从而引入新的概念抽象，而这一新的概念抽象一定会使集群服务管理和使用更容易扩展和重用。

# k8s主要特性：
- **自动化部署**
应用部署到容器中，kubernetes能够根据应用程序的计算资源需求和其它一些限制条件，自动地将运行应用程序的容器调度到集群中指定的Node上，在这个过程中并不会影响应用程序的可用性
- **系统自愈**
当Kubernetes集群中某些容器失败时，会重新启动他们。当某些Node挂掉后，Kubernetes会自动重新调度这些Node上容器到其他可用的Node上。如果某些容器没有满足用户定义的健康检查条件，这些容器会被判定为无法正常工作的，集群会自动kill掉这些容器，在这个过程中直到容器被重新启动或重新调度，直到可用以后才会对调用的客户端可见。
- **水平扩展**
在Kubernetes中，通过一个命令就可以实现应用程序的水平扩展，实现这个功能的是HPA（Horizontal Pod Autoscaler）对象。HPA是通过Kubernets API Resource和Controller的方式实现的，其中Resource决定了Controller的行为，而Controller周期性地调整Replication Controller或Deployment中的副本数，使得观察到的平均CPU使用情况与用户定义的能够匹配。
- **服务发现和负载均衡**
Kubernetes内置实现了服务发现的功能，他会给每个容器指派一个IP地址，给一组容器指派一个DNS名称，通过这个就可以实现服务的负载均衡功能。
- **自动更新和回滚**
当我们开发的应用程序发送变更，Kubernetes可以实现滚动更新，同时监控应用的状态，确保不会在同一时刻杀掉所有的实例，造成应用在一段时间范围内不可用。如果某些时候应用更新出错，Kubernetes能够自动地将应用恢复到原来正确的状态。
- **密钥和配置管理**
Kubernetes提供了一种机制（ConfigMap），能够使我们的配置数据与应用对应的Docker镜像解耦合，如果配置需要变更，不需要重新构建Docker镜像，这为应用开发部署提供很大的灵活性。
同时，对于应用所依赖的一些敏感信息，如用户名和密码、令牌、秘钥等信息，Kubernetes也通过Secret对象实现了这些敏感配置信息与应用的解耦合，这对应用的快速开发和交付提供便利，也提供了安全的保障
- **存储挂载**
Kubernetes可以支持挂载不同类型存储系统，比如本地存储、公有云存储（如AWS、GCP）、网络存储系统（如NFS、ISCSI、Gluster、Ceph、Cinder、Flocker）等，我们可以进行灵活的选择。
- **批量作业执行**
Kubernetes也支持批量作业的处理、监控、恢复。作业提交以后，直到作业运行完成即退出。如果运行失败，Kubernetes能够使失败的作业自动重新运行，直到作业运行成功为止。


# 组件说明
## Master 组件
Master组件提供集群的管理控制中心。
Master组件可以在集群中任何节点上运行
### kube-apiserver
kube-apiserver用于暴露Kubernetes API。任何的资源请求/调用操作都是通过kube-apiserver提供的接口进行.
### etcd
etcd是kubernetes提供默认的存储系统，保存所有集群数据，使用时需要为etcd数据提供备份计划
### kube-controller-manager
kube-controller-manager运行管理控制器，它们是集群中处理常规任务的后台线程。逻辑上，每个控制器是一个单独的进程，但为了降低复杂性，它们都被编译成单个二进制文件，并在单个进程中运行
这些控制器包括：
- 节点（Node）控制器
- 副本（Replication）控制器：负责维护系统中每个副本中的pod
- 断点（Endpoints）控制器：填充Endpoints对象（即连接Services & Pods）
- Service Account和Token控制器：为新的Namespace创建默认账号访问API Token

### cloud-controller-manager
云控制器管理器负责与底层云提供商的平台交互。云控制管理器是Kubernetes版本1.6中引入的，目前还是Alpha的功能

云控制器管理器仅运行云提供商特定的（controller loops）控制器循环。可以通过将--cloud-provider flag设置为external启动kube-controller-manager ，来禁用控制器循环。

cloud-controller-manager 具体功能：
- 节点（Node）控制器
- 路由（Route）控制器
- Service控制器
- 卷（Volume）控制器

### kube-scheduler
kube-scheduler监视新创建没有分配到Node的Pod，为Pod选择一个Node
### 插件 addons
插件是实现集群pod和Service功能的。Pod由Deployments，ReplicationController等进行管理。Namespace插件对象是在kube-system Namespace中创建

#### DNS
虽然不严格要求使用插件，但Kubernetes集群都应该具有集群DNS
集群DNS是一个DNS服务器，能够为Kubernetes services提供DNS记录
由Kubernetes启动的容器自动将这个DNS服务器包含在它们的DNS searches中
#### 用户界面
kube-ui提供集群状态基础信息查看
#### 容器资源监测
容器资源监控提供一个UI浏览器监控数据
#### Cluster-level Logging
Cluster-level logging，负责保存容器日志，搜索/查看日志。

## 节点（Node）组件
### kubelet
kubelet是主要的节点代理，它会监视已分配给节点的pod，具体功能：
- 安装Pod所需的volume。
- 下载Pod的Secrets。
- Pod中运行的 docker（或experimentally，rkt）容器。
- 定期执行容器健康检查。
- 如果必要的话，可以通过创建镜像pod来将pod的状态反馈到系统的其他部分。
- 将节点的状态报告给系统的其余部分。

### kube-proxy
kube-proxy通过在主机上维护网络规则并执行连接转发来实现Kubernetes服务抽象。
### docker
docker用于运行容器。
### RKT
rkt运行容器，作为docker工具的替代方案。
### supervisord
supervisord是一个轻量级的监控系统，用于保障kubelet和docker运行。
### fluentd
fluentd是一个守护进程，可提供cluster-level logging.。



