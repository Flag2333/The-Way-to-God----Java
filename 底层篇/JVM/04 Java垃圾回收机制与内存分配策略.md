## Java垃圾回收机制与内存分配策略

#### 1. 垃圾回收机制回收的是哪里的什么东西？

​	垃圾回收机制回收的是堆里的已经死去的对象

#### 2. 怎么判断对象已经死去？

- ##### 引用计数算法

  - 原理：给对象添加一个计数器，当这个对象被引用一次的时候，计数器+1，失去一个引用的时候-1，当计数器为0的时候表示没有被引用，也就是可以被垃圾收集器收集。
  - 缺点：很难解决对象之间相互循环引用问题

- ##### 可达性分析算法

  - 原理：GC Roots 作为起始点，从这些节点开始向下搜索，走过的路径为引用链，一个对象和起始点没有引用链链接的话说明这个对象可以被垃圾收集器收集
  - 可作为GC Roots的对象：
    1. 虚拟机栈（栈帧中的本地变量表）中引用的对象
    2. 方法区中类静态属性引用的对象
    3. 方法区中常量引用的对象
    4. 本地方法栈中JNI（即一般说的Native方法）引用的对象
  - 缺点：非黑即白

- ##### 引用

  - 强引用：“Object obj = new Object()”这种，垃圾收集器绝不会回收
  - 软引用：只有内存不够了才会回收，如果回收后还不够会报内存溢出异常
  - 弱引用：只要垃圾收集器工作，弱引用的对象就会被回收
  - 虚引用：为对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知

#### 3. 怎么清除已经死去的对象（垃圾收集算法）？

- ##### 标记-清除算法

  - 原理：把待回收的对象进行标记，之后统一回收
  - 缺点：
    1. 效率低，标记和清除效率都不高
    2. 空间问题，产生空间碎片，之后需要存储大对象的时候内存不足会引发下一次垃圾回收

- ##### 复制算法

  - 原理：每次只用一半内存，满了就把不需要回收的复制到另外内存，之后回收全部之前的内存
  - 缺点：每次只用一半有点小
  - 改善：把内存分为一块较大的Eden空间和两块较小的Survivor空间，每次只用Eden和一块Survivor，回收的时候把他们两块上还活着的都复制到另一块Survivor上，清理掉之前两块。Eden：Survivor = 8 : 1

- ##### 标记-整理算法

  - 原理：把需要回收的对象标记，把不需要回收的对象整理到一端，之后回收端边界以外的内存

- ##### 分代收集法

  - 原理：把堆分成新生代和老年代，新生代用复制算法，老年代用标记-清理或标记-整理算法

#### 4. 垃圾收集器

- ##### 有哪些垃圾收集器？

  1. Serial 收集器
  2. ParNew 收集器
  3. Parallel Sccavenge 收集器
  4. Serial Old 收集器
  5. Parallel Old 收集器
  6. CMS 收集器
  7. G1 收集器

- ##### G1收集器

  - 特点：
    1. 并行与并发：使用并发来缩短停顿时间
    2. 分代收集
    3. 空间整合：减少内存空间碎片
    4. 可预测的停顿：可以有计划的避免在Java堆中进行全区域的垃圾收集
  - 运行步骤：
    1. 初始标记
    2. 并发标记
    3. 最终标记
    4. 筛选回收

#### 5. 内存分配策略

- ##### 对象优先在Eden上分配

  Eden上内存不足时将触发一次垃圾回收

- ##### 大对象直接进入老年代

  避免短命大对象

- ##### 长期存活对象将进入老年代

  每熬过一次垃圾收集，对象年龄增加一岁，增加到15岁（默认为15）会被晋升到老年代