
# Mesos 架构

![Mesos 架构](http://mesos.apache.org/assets/img/documentation/architecture3.jpg)

上面的示意图展示了Mesos的主要组件。Mesos由一个*主*daemon，和其管理着的运行在每个集群节点上的*从*daemon，以及运行在*从*daemon中的*任务*的*mesos应用程序* （也称之为*框架*）三部分组成。

主daemon激活跨应用的细粒度资源(CPU，内存，...)共享，将之作为*资源提供者*。每个资源提供者包含一个<slave ID, resource1: amount1, resource2, amount2, ...>的列表。主daemon决定*多少*资源提供给每个框架，方法是通过给定的组织规则，诸如公平共享，或者是严格的优先级。要支持一组不同的策略的话，由于主daemon支持模块化的体系结构,所以通过插件机制是非常方便的添加一个新的分配模块的。


A framework running on top of Mesos consists of two components: a *scheduler* that registers with the master to be offered resources, and an *executor* process that is launched on slave nodes to run the framework's tasks (see the [App/Framework development guide](app-framework-development-guide.md) for more details about application schedulers and executors). While the master determines **how many** resources are offered to each framework, the frameworks' schedulers select **which** of the offered resources to use. When a frameworks accepts offered resources, it passes to Mesos a description of the tasks it wants to run on them. In turn, Mesos launches the tasks on the corresponding slaves.

## 资源提供者的实例

下面示意图展示了一个框架如何获得调度去运行一个任务的实例。

![Mesos Architecture](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

让我们来看下示意图中发生的事件。

1. Slave 1 reports to the master that it has 4 CPUs and 4 GB of memory free. The master then invokes the allocation policy module, which tells it that framework 1 should be offered all available resources.
1. The master sends a resource offer describing what is available on slave 1 to framework 1.
1. The framework's scheduler replies to the master with information about two tasks to run on the slave, using <2 CPUs, 1 GB RAM> for the first task, and <1 CPUs, 2 GB RAM> for the second task.
1. Finally, the master sends the tasks to the slave, which allocates appropriate resources to the framework's executor, which in turn launches the two tasks (depicted with dotted-line borders in the figure). Because 1 CPU and 1 GB of RAM are still unallocated, the allocation module may now offer them to framework 2.

In addition, this resource offer process repeats when tasks finish and new resources become free.

While the thin interface provided by Mesos allows it to scale and allows the frameworks to evolve independently, one question remains: how can the constraints of a framework be satisfied without Mesos knowing about these constraints? For example, how can a framework achieve data locality without Mesos knowing which nodes store the data required by the framework? Mesos answers these questions by simply giving frameworks the ability to **reject** offers. A framework will reject the offers that do not satisfy its constraints and accept the ones that do.  In particular, we have found that a simple policy called delay scheduling, in which frameworks wait for a limited time to acquire nodes storing the input data, yields nearly optimal data locality.

你也可以通过阅读[技术白皮书](http://mesos.berkeley.edu/mesos_tech_report.pdf)来进一步的了解Mesos体系结构。
