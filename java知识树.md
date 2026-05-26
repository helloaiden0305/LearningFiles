# Java 知识树

```
Java
│
├── 一、面向对象基础
│   ├── 语言对比
│   │   └── C++和java的区别（内存/指针/跨平台/继承）
│   ├── 核心特性
│   │   ├── 抽象类与接口的区别和共同点（is-a vs can-do）
│   │   ├── 接口中default方法和static方法的区别和使用场景
│   │   ├── 多态怎么理解（重载看左边 → 重写看右边）
│   │   └── final、static、synchronized（限制 / 归属类 / 并发安全）
│   └── 拷贝
│       └── 深拷贝与浅拷贝（引用共享 vs 独立副本 / clone native方法）
│
├── 二、数据类型与字符串
│   ├── 基本类型 vs 包装类
│   │   ├── 有了基本类型为什么要包装类？自动装拆箱及频繁装箱的影响
│   │   └── Integer缓存池机制（-128~127 / IntegerCache / -XX:AutoBoxCacheMax）
│   ├── String
│   │   ├── String、StringBuffer、StringBuilder 的区别
│   │   ├── String 不可变的原因（安全 / 常量池 / 线程安全）
│   │   └── new String("ABC")创建了几个对象？
│   └── 对象比较
│       ├── equals 与 ==、hashCode
│       └── 如何重写equals()方法？为什么还要重写hashCode()
│
├── 三、集合框架（JCF）
│   ├── 总览
│   │   └── 谈谈 List、Set、Map 的区别
│   ├── List（有序可重复）
│   │   ├── ArrayList 和 LinkedList 的区别
│   │   ├── ArrayList 的扩容机制（1.5倍 / 懒加载 / 默认容量10）
│   │   ├── 为什么不用按需扩容？倍数扩容不会有内存浪费吗？
│   │   └── CopyOnWriteArrayList的实现原理及适用场景（弱一致性 / 读多写少）
│   ├── Map
│   │   ├── HashMap 与 HashTable 的区别
│   │   ├── HashMap 的实现原理？可能出现的问题？
│   │   │   ├── 数组+链表/红黑树，负载因子0.75
│   │   │   ├── 链表转红黑树阈值=8，树转链表阈值=6
│   │   │   └── 扩容时死循环（JDK7头插法 → JDK8尾插法）
│   │   ├── LinkedHashMap 的工作原理（双向链表保序 / LRU缓存）
│   │   └── ConcurrentHashMap
│   │       ├── 谈谈对 ConcurrentHashMap 的理解（JDK7分段锁 → JDK8 CAS+sync）
│   │       ├── 扩容和 HashMap 相比（多线程协作搬迁 / 转发节点）
│   │       ├── 为什么不允许Key或Value为null？（消灭二义性）
│   │       ├── size()如何保证高并发下统计准确性？（baseCount+CounterCells近似值）
│   │       └── 能保证完全线程安全吗？（单操作安全 vs 复合操作不安全）
│   ├── Set
│   │   └── TreeMap和TreeSet的原理（红黑树 / TreeSet套皮TreeMap）
│   └── Queue
│       ├── ArrayBlockingQueue与LinkedBlockingQueue有何区别？怎么选
│       ├── PriorityQueue的底层数据结构？如何维持堆属性？（小顶堆 / 2i+1/2i+2）
│       └── DelayQueue的实现原理及其在定时任务中的应用（Leader-Follower）
│
├── 四、异常处理
│   ├── Error 与 Exception
│   ├── Java中异常分哪两类？有什么区别？（受检 vs 非受检）
│   └── try_catch_finally中的finally代码块一定会执行吗？
│
├── 五、高级特性
│   ├── 反射机制？性能开销？如何优化？
│   ├── 静态代理与动态代理的区别？JDK动态代理与CGLIB动态代理有何不同
│   ├── 泛型类型擦除及局限性？为什么不能创建泛型数组？
│   ├── Lambda表达式的底层实现原理（invokedynamic + LambdaMetafactory）
│   └── Java的注解是什么（元数据标签，逻辑在读它的人）
│
└── 六、多线程与并发
    ├── 线程基础
    │   ├── 为什么要使用多线程
    │   ├── 线程安全怎么理解？有哪些实现方式？
    │   ├── Java中使用多线程的方式有哪些？/ 启动线程的方式
    │   └── Thread 中 run() 与 start() 的区别
    │
    ├── 线程状态
    │   ├── 说一下线程的几种状态？（NEW/RUNNABLE/BLOCKED/WAITING/TIMED_WAITING/TERMINATED）
    │   ├── 线程sleep()和wait()的区别？
    │   └── 线程阻塞的原因？（等锁/等条件/IO/死锁）
    │
    ├── 线程池
    │   ├── 线程池的状态有哪些？
    │   │   └── RUNNING → SHUTDOWN → STOP → TIDYING → TERMINATED
    │   └── 为什么不推荐使用 Executors.newFixedThreadPool()？
    │       └── 无界队列 → OOM，推荐直接用 ThreadPoolExecutor
    │
    ├── 同步与锁
    │   ├── 如何实现多线程中的同步？（锁 / 原子类 / 工具类 / volatile）
    │   ├── 说说你对 synchronized 的了解
    │   ├── synchronized的原理是什么？（对象头Mark Word / Monitor / 字节码指令）
    │   ├── Synchronized 的锁升级（偏向 → 轻量 → 重量）
    │   ├── synchronized和volatile关键字的区别？
    │   ├── 线程死锁及如何避免？（四必要条件 / 破坏一个即可）
    │   ├── notify 和 notifyAll有什么区别？
    │   └── Volatile 关键词的作用（可见性 + 有序性）
    │
    └── ThreadLocal
        ├── ThreadLocal用法和原理（线程私有副本 / 用空间换锁）
        └── ThreadLocal 为什么会造成内存泄漏（弱引用key / 强引用value / 用后remove）
```
