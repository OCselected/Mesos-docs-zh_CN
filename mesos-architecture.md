
# Mesos 架构

![Mesos 架构](http://mesos.apache.org/assets/img/documentation/architecture3.jpg)

上面的示意图展示了Mesos的主要组件。Mesos由一个*主*daemon，和其管理着的运行在每个集群节点上的*从*daemon，以及运行在*从*daemon中的*任务*的*mesos应用程序* （也称之为*框架*）三部分组成。

主daemon激活跨应用的细粒度资源(CPU，内存，...)共享，将之作为*资源提供者*。每个资源提供者包含一个<slave ID, resource1: amount1, resource2, amount2, ...>的列表。主daemon决定*多少*资源提供给每个框架，方法是通过给定的组织规则，诸如公平共享，或者是严格的优先级。要支持一组不同的策略的话，由于主daemon支持模块化的体系结构,所以通过插件机制是非常方便的添加一个新的分配模块的。

Mesos 框架运行在最上层，由两部分组成: *调度器*注册到主daemon以申请资源，*执行者*进程会在从节点启动以运行框架的任务（参考[框架开发指南](http://docs.ocselected.org/mesos-docs/app-framework-development-guid.hmtl)获得更多关于应用程序调度器和执行者的细节)。当主daemon决定**多少**资源给每个框架，框架的调度器选择**哪个**所提供者来使用。当框架接受了提供的资源，它会传递到Mesos所描述的打算运行它们的任务中。反过来，Mesos在对应的从节点启动任务。

## 资源提供者的实例

下面示意图展示了一个框架如何获得调度去运行一个任务的实例。

![Mesos Architecture](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

让我们来看下示意图中发生的事件。

1. 从节点1 报告给主daemon它拥有4 CPU和4GB的可用内存。主daemon然后调用分配策略模块，告诉它,框架1应得到所有可用资源。
1. 主daemon发送给框架1的信息是：描述了在从节点1中有多少可用的资源。
1. 框架调度器回复给主daemon的信息是关于运行在从节点的两个任务的信息，第一个任务使用了<2 CPUs, 1 GB RAM>，第二个任务使用了<1 CPUs, 2 GB RAM>。
1. 最后，主daemon发送任务给从节点，分配合适的资源给框架的执行者，执行者会启动两个任务(图中描绘的虚线边框)。因为1 CPU和1GB内存还是未分配，分配模块现在可以将它们提供给框架2.

另外，此资源提供的过程，在任务结束且新的资源被释放时，重新来过。

Mesos提供了轻量的接口，允许它扩展且允许框架独立的处理，那么问题来了:在Mesos不知道的约束的情况下约束如何满足框架？举例来说，框架如何实现局部性？且还是其自身所需然而Mesos又不知道数据存放在那个节点的情况下。Mesos通过简单的赋予框架**拒绝**提供的能力很好的回答了这个问题。一个框架会拒绝提供不满足约束，接受满足约束的。特别是,我们发现一个简单的策略称为延迟调度,在框架等待的有限的时间内获取节点存储的输入数据,收益率近最优数据本地化。

你也可以通过阅读[技术白皮书](http://mesos.berkeley.edu/mesos_tech_report.pdf)来进一步的了解Mesos体系结构。
