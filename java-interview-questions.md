# Java 面试题库

## Java 基础

1. C++ 和 Java 的区别是什么？
2. 抽象类与接口的区别和共同点是什么？
3. 接口中引入了 default 方法和 static 方法，请说明它们的区别和使用场景。
4. 多态怎么理解？
5. 有了基本类型为什么要包装类？自动装拆箱是什么？频繁装箱有啥影响？
6. Integer 的缓存池机制是如何工作的？范围是多少？能否调整？
7. final、static、synchronized 的区别？
8. String、StringBuffer、StringBuilder 的区别？
9. String 不可变的原因是什么？
10. `new String("ABC")` 创建了几个对象？
11. `equals` 与 `==` 的区别？
12. 如何重写 `equals()` 方法？为什么还要重写 `hashCode()`？
13. 深拷贝与浅拷贝的底层原理是什么？区别在哪里？
14. 序列化和反序列化的作用是什么？
15. Error 与 Exception 的区别？
16. Java 中异常分哪两类？有什么区别？
17. 反射机制？它会带来哪些性能开销？如何优化？
18. 静态代理与动态代理的区别？JDK 动态代理与 CGLIB 动态代理有何不同？
19. 泛型类型擦除并说说其局限性？为什么不能创建泛型数组？
20. Lambda 表达式的底层实现原理是什么？
21. Java 的注解是什么？
22. Java 中 try-catch-finally 中的 finally 代码块一定会执行吗？

## Java 集合

23. 谈谈 List、Set、Map 的区别？
24. 谈谈 ArrayList 和 LinkedList 的区别？
25. 请说一下 HashMap 与 Hashtable 的区别？
26. 谈一谈 ArrayList 的扩容机制？为什么不用按需扩容？倍数扩容会有内存浪费吗？
27. CopyOnWriteArrayList 的实现原理及适用场景？为什么它是"弱一致性"的？
28. HashMap 的底层实现原理？可能出现的问题是什么？
29. JDK8 中 HashMap 链表转红黑树的阈值是多少？为什么是 8？
30. 请简述 LinkedHashMap 的工作原理和使用方式？
31. 谈谈你对 ConcurrentHashMap 的理解？
32. ConcurrentHashMap 的扩容和 HashMap 相比有什么差异？
33. 为什么 ConcurrentHashMap 不允许 Key 或 Value 为 null？
34. ConcurrentHashMap 的 `size()` 方法是如何保证高并发下的统计准确性的？
35. ConcurrentHashMap 能保证完全线程安全吗？
36. ArrayBlockingQueue 与 LinkedBlockingQueue 有何区别？怎么选？
37. PriorityQueue 的底层数据结构是什么？它是如何维持堆属性的？
38. DelayQueue 的实现原理及其在定时任务中的应用？
39. TreeMap 和 TreeSet 的底层原理是怎样的？

## Java 多线程

40. 为什么要使用多线程？
41. 线程安全怎么理解？有哪些实现方式？
42. Java 的多线程开发中，线程数是不是越多越好？一般怎么设置？
43. Java 中什么是线程池？核心矛盾是什么？
44. 线程池的核心参数有哪些？
45. Java 线程池的拒绝策略有哪些？
46. volatile 关键字的作用是什么？
47. 如何理解 JMM（Java 内存模型）？
48. Java 中使用多线程的方式有哪些？
49. 请谈谈 Thread 中 `run()` 与 `start()` 的区别？
50. 线程池的状态有哪些？
51. 为什么不推荐使用 `Executors.newFixedThreadPool()`？
52. 说一下线程的几种状态？
53. 线程 BLOCKED 和 WAITING / TIMED_WAITING 有什么区别？
54. 谈一谈线程 `sleep()` 和 `wait()` 的相同点和区别？
55. Java 线程中 `notify()` 和 `notifyAll()` 有什么区别？
56. 如何实现多线程中的同步？
57. 对 synchronized 怎么理解？
58. synchronized 的底层原理是什么？
59. 什么是 synchronized 的锁升级？
60. 线程死锁产生的四个必要条件？如何有效地避免？
61. 谈谈线程阻塞的原因？
62. synchronized 和 volatile 关键字的区别？
63. AQS 的核心机制和应用场景是什么？
64. 谈谈 ThreadLocal 的用法和原理？为什么会造成内存泄漏？
65. 什么是 happens-before？
66. 什么是 CAS？什么是 ABA 问题？如何解决？
67. AtomicInteger 适合高并发，但在竞争极其激烈时性能会下降，为什么？有什么优化方案？

## JVM

68. Java 是编译型还是解释型语言？
69. 什么是 JIT？
70. 如何理解 JVM 的内存逃逸（Escape Analysis）？
71. JVM 中堆和栈有什么区别？
72. Java 对象的内存布局是怎样的？
73. JVM 的运行内存区域有哪些？
74. 永久代为什么要改成元空间？
75. 字符串常量池为什么搬到堆内存？
76. JVM 类加载的过程是怎样的？
77. 双亲委派的过程是怎样的？目的是什么？
78. 什么情况下需要打破双亲委派？
79. JVM 是如何判断一个对象是可回收的？
80. 如何判断一个类可回收？常量是废弃常量吗？
81. Java 中的强引用、软引用、弱引用、虚引用有什么区别？
82. 常见的垃圾收集算法有哪些？
83. 为什么新生代用复制算法？老年代不用？
84. 什么是 STW（Stop-The-World）？
85. 常见的垃圾收集器有哪些？
86. G1 对比 CMS 优势在哪？
87. GC 频繁有可能是什么原因造成的？
88. JVM 和 Android ART 在 GC 方面的差异有哪些？
