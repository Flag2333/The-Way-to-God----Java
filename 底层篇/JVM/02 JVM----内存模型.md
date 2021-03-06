## Java内存模型（JMM）

- #### 概念：

  > 是一种抽象的概念，并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。 

- #### 作用：

  JVM运行是以线程为单位的，JMM是负责控制把主内存中的变量拷贝到每个线程单独的工作内存中（用于线程私有），并且再负责把每个线程的变量同步到主内存。

  > 线程之间通过写-读内存中的公共状态来隐式进行通信 。

  ![线程，主内存，工作内存三者交互(基于JMM)](https://github.com/Flag2333/TheWayToGod-Java/blob/master/img/%E7%BA%BF%E7%A8%8B%EF%BC%8C%E4%B8%BB%E5%86%85%E5%AD%98%EF%BC%8C%E5%B7%A5%E4%BD%9C%E5%86%85%E5%AD%98%E4%B8%89%E8%80%85%E4%BA%A4%E4%BA%92%EF%BC%88%E5%9F%BA%E4%BA%8EJMM%EF%BC%89.png)

- #### 必要性：

  - 解决每个线程向主内存同步数据时造成的线程安全问题。

  - JMM是围绕着程序执行的原子性、有序性、可见性展开的 , JMM保证这三个特性。

    ​	JMM提供的解决方案：

    > 1. 工作内存与主内存**同步延迟现象导致的可见性问题**，可以使用**synchronized**关键字或者**volatile**关键字解决，它们都可以使一个线程修改后的变量立即对其他线程可见。 
    > 2. 对于**指令重排导致的可见性问题和有序性问题**，则可以利用**volatile**关键字解决，因为volatile的另外一个作用就是禁止重排序优化。
    > 3. 除了靠sychronized和volatile关键字来保证原子性、可见性以及有序性外，JMM内部还定义一套**happens-before 原则**来保证多线程环境下两个操作间的原子性、可见性以及有序性。 
    >

- #### 内存可见性

  - ##### 概念：

    > 可见性指的是当一个线程修改了某个共享变量的值，其他线程是否能够马上得知这个修改的值。 

  - ##### 造成内存可见性问题的原因

    1. 工作内存与主内存同步延迟
    2. 指令重排以及编译器优化 

  - ##### 解决方式

    1. synchronized关键字：线程解锁前把共享变量同步到主内存，加锁时清空工作内存中的共享变量的值，从主内存中重新获取共享变量的值。
    2. volatile关键字：读volatile变量时都重新从主内存同步共享变量；写volatile变量，当变量发生改变后，强制把变量同步到主内存。（读和写是两步，可以拆开----不能保证原子性。解决方法：synchronized）

- #### 重排序

  - ##### 概念：

    代码执行的顺序与编写代码的顺序不一定相同

  - ##### 作用：

    提高性能，提高效率

  - ##### 分类：

    1. 编译器优化的重排
    2. 指令并行的重排 
    3. 内存系统的重排 

  - ##### as-if-serial：

    无论如何重排序，程序执行的结果与按照代码顺序执行的结果一致（单线程下默认遵循as-if-serial语义）

- #### 顺序一致性

  - ##### 特性：

    > 1. 一个线程中的所有操作必须按照程序的顺序来执行。
    > 2. （不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。

- #### volatile

  - 能保证内存可见性，不能保证原子性
  - 使用场景：
     1. 对变量的写入操作不依赖其当前值
     2. 该变量没有包含在具有其他变量的不变式中
  - 无需加锁，比synchronized更轻量级，效率高于synchronized

- #### final 

  用来修饰类，方法，变量等，被final修饰的类不可以继承扩展，变量不可以修改，方法不可以重写。是保证平台安全的必要手段。 

- #### PS：

  - 对64位（long,double）变量的读写可能不是原子操作，有可能分为两次32位的读写操作。  解决方式：volatile

- #### 参考资料

  - [全面理解Java内存模型(JMM)及volatile关键字 ](https://blog.csdn.net/javazejian/article/details/72772461)
  - [深入理解Java内存模型（一）——基础](http://www.infoq.com/cn/articles/java-memory-model-1)
  - [深入理解Java内存模型（三）——顺序一致性](http://www.infoq.com/cn/articles/java-memory-model-3)



