## JVM-HotSpot虚拟机-即时编译器、编译优化 

#### 1. 解释器与编译器相对于被执行代码的关系

![解释器与JIT编译器](https://github.com/Flag2333/TheWayToGod-Java/blob/master/img/%E8%A7%A3%E9%87%8A%E5%99%A8%E4%B8%8EJIT%E7%BC%96%E8%AF%91%E5%99%A8.png)

#### 2. 为何HotSpot虚拟机要使用解释器与编译器并存的架构

``` bash
在不同的情况下需要使用不同的去执行
1. 程序需要快速启动时，使用编译器，节省时间
2. 多次运行后，热点代码编译成本地代码，提高效率
3. 用解释执行节约内存，用编译执行提高效率
4. 解释器是编译器激进优化的逃生门，优化不成功则使用解释器继续执行
```

#### 3. 为何HotSpot虚拟机要实现两个不同的即时编译器

- 分为哪两个即时编译器

  ``` bash
  Client Complier : C1编译器
  Server Complier : C2编译器
  ```

- 原因：可以平衡响应速度与运行效率更好的优化代码

  ``` bash
  为了让编译器更好的优化代码，解释器还要替编译器收集性能监控信息，此举耗费时间；为了更好的平衡响应速度与运行效率，采用“分层编译”策略
  1. 0级：不开启监控，可触发1
  2. 1级：采用C1编译器进行简单可靠优化，如果有必要则监控
  3. 2级或以上：采用C2编译器进行耗时优化，根据性能监控进行激进优化
  ```

#### 4. 程序何时使用解释器执行？何时使用编译器执行？

答案同1

#### 5. 哪些程序代码会被编译为本地代码？如何编译为本地代码？

- 哪些程序代码会被编译为本地代码？

  ```bash
  热点代码
  ```

- 哪些是热点代码？

  ```bash
  1. 被多次调用的方法（方法调用计数器触发）
  2. 被多次执行的循环体(回边计数器触发)
    - 栈上替换（OSR）：是一种能够在非方法入口处进行解释执行和编译后代码之间切换的技术。OSR 编译可以用来解决单次调用方法包含热循环的性能优化问题。
  ```

- 热点代码怎么探测？

  ```bash
  - 基于采样的热点探测：周期性的检查
  - 基于计数器的热点探测：设置计数器，默认阈值为1500
  	1. 方法调用计数器：统计的是这段代码在一段时间内执行的相对次数（有计数热度衰减）
  	2. 回边计数器：统计的是方法体中循环代码的回边次数（没有计数热度衰减，则统计的是绝对次数）
  ```

- 编译过程

  C1与C2编译器执行过程不同

  - C1编译器

    ![Client Complier 架构](https://github.com/Flag2333/TheWayToGod-Java/blob/master/img/Client%20Complier%20%E6%9E%B6%E6%9E%84.jpg)

    1. HIR：高级中间代码 作用：可以使得一些在HIR的构造过程之中和之后进行的优化动作更容易实现
    2. LIR：低级中间代码（上边LTR应该为LIR）

  - C2编译器

    会执行所有经典的优化动作

    - 无用代码消除
    - 循环展开
    - 循环表达式外提
    - 消除公共子表达式
    - 常量传播
    - 基本快重排序
    - 检查消除
    - 空值检查消除
    - 守护内联
    - 分支频率预测等

  - 采用分层编译模式（Java7 之后）综合了 C1 的启动性能优势和 C2 的峰值性能优势。 

    1. 解释执行；
    2. 执行不带 profiling（性能监控功能） 的 C1 代码；
    3. 执行仅带方法调用次数以及循环回边执行次数 profiling 的 C1 代码；
    4. 执行带所有 profiling 的 C1 代码；
    5. 执行 C2 代码。 

    - 通常情况下，C2 代码的执行效率要比 C1 代码的高出 30% 以上。
    - 然而，对于 C1 代码的三种状态，按执行效率从高至低则是 1 层 > 2 层 > 3 层。其中 1 层的性能比 2 层的稍微高一些，而 2 层的性能又比 3 层高出 30%。
    - 这是因为 profiling 越多，其额外的性能开销越大。 

    ps：profiling 是指在程序执行过程中，收集能够反映程序执行状态的数据。这里所收集的数据我们称之为程序的 profile。 

#### 6. 编译优化技术

- 语言无关的经典优化技术之一：公共子表达式消除

  ```bash
  如果一个表达式E已经被计算过了，并且从先前计算到现在，E中所有的变量都没有发生变化，E的这次出现就成为了公共字表达式
  ```

- 语言相关的经典优化技术之一：数组范围检查消除

  ```bash
  检查数组是否数组越界，是否空指针，是否除数为零等等
  ```

- 最重要的优化技术之一：方法内联

  ```bash
  - 方法内联：在编译过程中，当遇到方法调用时，将目标方法的方法体纳入编译范围之中，并取代原方法调用的优化手段。
  - 优化表现：把目标方法的代码“复制”到发起调用的方法之中，避免发生真实的方法调用
  - 作用：1. 消除方法调用成本
  	   2. 为其他优化手段建立良好基础
  - 对于Java的多态，在方法内联的时候并不知道对象的实际类型是什么，这种情况会进行“类型继承关系分析”
  	1. 查询当前程序下是否有多个方法可供选择，如果只有一个，则进行内联（属于激进优化，需要“逃生门”，称为守护内联），如果后续执行过程中一直没有加载到使继承关系变化的类，则一直内联下去，如果加载到了，则退回到解释状态执行，或者重新进行编译
  	2. 如果有多个方法可以选择，则采用内联缓存的方式。
  		未调用方法时，缓存为空，调用后则缓存记录方法接收者的版本信息，并且每次调用都比较版本信息，如果相同则继续内联下去，如果不同则取消内联，查找虚方法表进行方法分派
  ```

- 最前沿的优化技术之一：逃逸分析

  ```bash
  - 即时编译器判断对象逃逸的依据有两个：
  	1. 看对象是否被存入堆中
  	2. 看对象是否作为方法调用的调用者或者参数。
  - 方法逃逸：当一个对象在方法中被定义后，他可能被外部方法所引用，例如作为调用参数传递到其他方法中。
  - 线程逃逸：对象被别的线程访问到，譬如复制给类变量或可以在其他线程中访问到
  - 为解决上述两个问题，优化方法
  	1. 栈上分配：经过逃逸分析，如果却定一个对象不会逃逸到方法外，可以在栈上分配内存，随栈帧出栈而销毁，减少在堆上分配内存还要进行GC的时间。
  	2. 同步消除：经过逃逸分析，如果确定这个变量不会被其他线程访问，这个变量的读写也就没有竞争，也就不用同步。
  	3. 标量替换：如果把一个Java对象拆散，根据程序访问的情况，将其使用到的成员变量恢复原始类型来访问就叫做标量替换。
  		经过逃逸分析，如果证明一个对象不会被外部访问，就可以不创建这个对象，只创建组成这个对象的若干的原始类型的成员变量。
  		优点：在栈上读写，为后续优化创造条件
  ```

