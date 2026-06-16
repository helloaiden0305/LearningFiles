## 基础概念

1. 说说协程
2. 协程具体是怎么实现的，Kotlin 实现协程的具体步骤
3. Java 内存模型
4. 方法中的变量在哪个位置，成员变量在哪个位置
5. 创建单例时怎么保证线程安全，懒汉式怎么保证线程安全
6. 答加锁，加锁会有阻塞等待，有更好的方法吗
7. 安卓中的内存分为哪些部分，PSS 和 RSS 的区别，Bitmap 位于什么部分内存中
8. 显示 Intent 和隐式 Intent 的区别
9. 启动服务的两种方式，除了生命周期还有什么区别
10. 安卓主线程为什么不能做耗时操作，会出现什么问题，超过多久会出问题
11. Activity 的生命周期
12. Fragment 生命周期中 `onCreateView` 和 `onViewCreated` 的区别
13. 安卓启动模式的 SingleTask 和 SingleTop 的区别及各自使用场景

## 内存与 GC

14. 内存泄漏有哪些具体的例子，循环引用的具体例子
15. 常见垃圾回收算法有哪些，对我们写代码有哪些警示
16. LeakCanary 的原理
17. 弱引用的底层原理
18. 怎么排查内存泄漏问题

## 并发

19. ConcurrentHashMap 的底层原理
20. synchronized 和 volatile 的区别

## 网络

21. TCP 的复用是怎么做的
22. HTTP 1.1 和 HTTP 2.0 连接复用的区别
23. OkHttp 的连接池是什么，怎么用的
24. 电量和网络对 App 的影响有了解过吗

## Handler 与线程

25. Handler 的 `postDelay` 为什么切到后台就不生效了
26. Handler 内存泄漏问题

## View 与 UI

27. View 的绘制流程
28. View 事件分发机制
29. 自定义 View 要重写哪些方法
30. MotionEvent 除了坐标、类型还有哪些常用的属性
31. 屏幕视图刷新原理，涉及哪些进程、组件
32. 怎么解决滑动冲突
33. 怎么排查掉帧问题，原因有哪些
34. 安卓中动画有哪些种类，插值器的实现
35. 悬浮窗怎么实现
36. 使用 `WindowManager.addView` 涉及哪些操作，假设两个窗口的层级一样会发生什么
37. 三指、四指操作怎么实现

## 架构

38. MVP 和 MVVM 的区别

## 项目与工程

39. 如果需要执行一些耗时的操作，你会怎么做
40. 安卓打出的包有哪些文件
    Android 打包生成的是 APK 文件，本质是一个 zip 压缩包，里面主要包含 
AndroidManifest.xml 声明组件和权限
代码编译成的 classes.dex
res 资源目录、assets 资源目录、resources.arsc、(各类资源目录、arsc资源索引R.id查询)
lib 动态库 以及 META-INF 签名信息等。
42. App 瘦身有哪些方式

44. 怎么用 AI 工具开发，需要注意什么
45. 了解 Skills 和 MCP 吗
