# 13. 并发编程

## 为什么要并发

并发是一种解耦策略，帮助我们分解：**做什么（目的）**和**何时（时机）**，改进应用程序的**响应时间**、**吞吐量** 和**结构**。

> 从结构角度来看，应用程序看起来更像是许多台协同工作的计算机，而不是一个大循环。系统因此会更易于被理解，给出了许多切分关注面的手段。



- 并发编程**很难**：
  - 并发会在性能和编写额外代码上**增加一些开销**；
  - **正确的并发是复杂的**，即便对于简单的问题也是如此；
  - 并发缺陷并非总能重现，所以常被看作偶发事件而忽略，未被当作真的缺陷看待；
  - **并发常常需要对设计策略的根本性修改**。



- 并发编程为什么很难：

因为线程在执行那行Java代码时有许多可能路径可行，有些路径会产生错误的结果。



## 并发防御原则

1. **单一权责原则 SRP**：

   - 并发相关代码有自己的开发、修改和调优生命周期；
   - 并发相关代码有自己要对付的挑战，和非并发相关代码不同，而且往往更加困难；
   - 写得不好的并发代码可能的出错方式数量非常多。

   > **建议**：分离并发相关代码和其他代码。



2. **限制数据作用域:**

   两个线程修改共享对象的同一字段时，可能互相干扰，导致未预期行为。

   **解决方案**：采用synchronized关键字在代码中保护一块使用共享对象的**临界区**。但是，需要限制临界区的数量。更新共享数据的地方越多：

   	- 越容易忘记保护临界区；
   	- 得多花力气保证一切都收到有效防护（破坏DRY原则）
   	- 很难找到错误源，也很难判断错误源。

   > **建议**：谨记数据封装；严格限制可能被共享的数据的访问。



3. **使用数据复本**：

   避免共享数据：

   	- 负责对象并以只读方式对待
   	- 从多个线程收集所有复本的结果，并在单个线程中合并这些结果。

   > 使用数据复本会增加创建额外对象的成本，但能避免代码同步执行。以内存换并发。



4. **线程应尽可能地独立**：

   不与其他线程共享数据，每个线程都像是唯一线程，没有同步需要。

   > **建议**：尝试将数据分解到可被独立线程（可能在不同处理器上）操作的独立子集。



## 了解Java库

1. 使用类库提供的线程安全群集；
2. 使用executor框架（executor framework）执行无关任务；
3. 尽可能使用非锁定解决方案；
4. 有几个类并不是线程安全的。



- 线程安全群集：

  JDK中java.util.concurrent包中的群集，对于多线程解决方案时安全的，执行良好；

  如果部署环境时Java 5，可以采用ConcurrentHashMap；

  支持高级并发设计的类：

![image-20190901151610077](/Users/zhushushu/Library/Application Support/typora-user-images/image-20190901151610077.png)



## 执行模型

基础定义：

![image-20190901151654780](/Users/zhushushu/Library/Application Support/typora-user-images/image-20190901151654780.png)



1. 生产者-消费者模型

   生产者线程创建某些工作，一个或多个消费者线程从队列中获取并完成这些工作。

2. 读者-作者模型

   为读者线程提供信息源，偶尔作者线程更新这些共享资源。

   吞吐量是一个问题：

   	- 增加吞吐量，会导致线程饥饿和过时信息的积累；
   	- 更新会影响吞吐量；
   	- 协调读者线程，不去读作者线程正在更新的信息，是一种辛苦的平衡工作；
   	- 作者线程长期锁定许多读者线程，将导致吞吐量问题。



## 警惕同步方法之间的依赖

synchronized可用来保护单个方法，但如果在同一个共享类中有多个同步方法，系统就可能写得不太正确了。

**建议**：避免使用一个共享对象的多个方法。

如果必须使用一个共享对象的多个方法：

- **基于客户端的锁定**：客户端代码在调用第一个方法前锁定服务器，确保锁的范围覆盖了调用最后一个方法的代码；
- **基于服务端的锁定**：在服务端内创建锁定服务端的方法，调用所有方法，然后解锁。让客户端代码调用新方法；
- **适配服务端**：创建执行锁定的中间层。这是一种基于服务端的锁定的例子，但不修改原始服务端代码。



## 保持同步区域微小

**synchronized**锁维护的所有代码区域在任一时刻保证只有一个线程执行。缺点：锁会带来延迟和额外开销。

临界区都需要被保护起来，所以临界区应尽可能少，但是不能通过扩大临界区来达到目的，否则会**增加资源争用、降低执行效率**。

> **建议**：尽可能减小同步区域。



## 很难编写正确的关闭代码

常见问题：**死锁**，线程一直等待永不会到来的信号。

> **建议**：尽早考虑关闭问题



## 测试线程代码

- 将伪失败看作可能的线程问题；

  伪失败：不可能失败的“失败”

  > **建议**：不要将系统错误归咎于偶发事件

- 先使非线程代码可工作；

  > **建议**：不要同时追踪非线程缺陷和线程缺陷

- 编写可插拔的线程代码；

  编写可在数个配置环境下运行的线程代码：

  - 单线程与多个线程在执行时不同的情况；
  - 线程代码与实物或测试替身互动；
  - 用运行快速、缓慢和有变动的测试替换执行；
  - 将测试配置为能运行一定数量的迭代。

- 编写可调整的线程代码；

  > **建议**：允许线程数量可调整。在系统运行时允许线程发生变动。允许线程依据吞吐量和系统使用率自我调整。

- 运行多于处理器数量的线程；

  任务交换越频繁，越可能找到错误临界区或导致死锁的代码。

- 在不同平台上运行；

  不同操作系统有着不同线程策略的事实，不同的线程策略影响了代码的执行。

- 调整代码并强迫错误发生。

  装置试错代码，触发偶发问题：增加对Object.wait()、Object.sleep()、Object.yield()、Object.priority()等方法的调用，改变代码执行顺序。

  两种装置代码的方法：

  - 硬编码；
  - 自动化。

  自动化实现方式：实现一个单方法类，并在代码不同位置调用这个方法，这个方法的有2种实现，在生产环境中什么都不做，在测试环境中，随机选择睡眠、让步或径直执行。

  ![image-20190901155824732](/Users/zhushushu/Library/Application Support/typora-user-images/image-20190901155824732.png)



## 总结：

1. 并发代码难写

2. 并发代码要诀：

   遵循**单一权责原则**：分离并发相关代码和其他代码；

   锁定区最小化；

   避免从锁定区域中调用其他锁定区域；

   尽可能减少共享对象和共享范围；

   向客户代码提供共享数据，而不是迫使客户代码管理共享状态。

3. 并发问题可能原因：
   - 对共享数据的多线程操作
   - 使用了公共资源池
   - 平静关闭或停止循环之类的边界问题



# 14. 逐步改进

本章是一个重构案例



编写整洁代码，必须先写肮脏代码，然后再**清理它**。

