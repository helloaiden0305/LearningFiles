# Android 积累

## 基础概念

### 1. 协程具体是怎么实现的，Kotlin 实现协程的具体步骤？

Kotlin 协程的实现本质是编译器把 `suspend` 函数转换成状态机。

每个 `suspend` 函数在编译后都会多一个 `Continuation` 参数，用来保存协程的执行状态。

- 当执行到挂起点时会保存当前状态并返回，线程被释放；
- 等任务完成后通过 `Continuation.resume()` 恢复执行，根据状态机继续运行。

利用状态机和参数，实现用户态（代码级别）的协程管理。

---

### 2. 方法中的变量在哪个位置，成员变量在哪个位置？

- **方法中的局部变量：** 在虚拟机栈的栈帧中的局部变量表里；
- **成员变量：** 属于对象，存储在堆中的对象实例中；
- **static 静态变量：** 属于类，在类加载时创建，存储在方法区（Java 8 之后是 Metaspace）。

---

### 3. 创建单例时怎么保证线程安全？

- **懒汉式单例：** 可以通过直接 `synchronized` 或双重检查锁保证线程安全，其中双重检查锁需要配合 `volatile` 防止指令重排序；
- **静态内部类方式：** 利用 JVM 类初始化阶段 `<clinit>()` 的同步机制，保证实例只会被创建一次，从而实现线程安全；
- **静态变量在外部类 → 饿汉式：** 首次类加载时创建单例。

---

### 4. 答加锁，加锁会有阻塞等待，有更好的方法吗？

如果直接使用 `synchronized` 或显式锁来保证线程安全，确实可能带来线程阻塞和上下文切换的开销。

可以使用 **CAS 等无锁机制、while 自旋、读写锁或减小锁粒度** 来提升并发性能。

---

### 5. 安卓中的内存分为哪些部分，PSS 和 RSS 的区别，Bitmap 位于什么部分内存中？

Android 内存主要包括 **3 + 3**：

- Java Heap、Native Heap、Stack、Code/Dex、共享内存和图形内存。

其中：

- **私有内存：** Java Heap、Native Heap、Stack
- **共享内存：** Code/Dex、Shared Memory、Graphics（通常通过 `mmap` 映射到进程地址空间，所以多个进程可以共享同一份物理内存）

**RSS 和 PSS** 都是衡量"某个进程"的内存占用指标：

- **RSS：** 表示进程实际占用的物理内存，会把共享内存重复计算；
  > `RSS ≈ Private + Code/Dex + Shared Memory + Graphics`

- **PSS：** 会按比例分摊共享内存，因此更能反映真实内存占用，Android 系统主要参考 PSS；
  > `PSS ≈ Private + (Code/Dex + Shared Memory + Graphics) / 进程数`

比如共享的共 40MB、4 个进程使用：

- RSS：每个进程算 40MB
- PSS：每个进程算 10MB

在 Android 8.0 之后，**Bitmap 的像素数据存储在 Native Heap，而 Bitmap 对象在 Java Heap。**

---

### 6. 显示 Intent 和隐式 Intent 的区别？

**指定方式不同：**

- **显示 Intent：** 明确指定要启动的组件类名（`ComponentName`）
- **隐式 Intent：** 不指定具体组件，只描述要执行的动作（action）和数据（data），由系统匹配

**使用场景不同：**

- **显示 Intent：** 一般用于应用内部组件之间的跳转（如 Activity → Activity）
- **隐式 Intent：** 用于调用系统组件或其他应用功能（如打开浏览器、分享、拨号）

**系统处理方式不同：**

- **显示 Intent：** 系统直接找到目标组件并启动
- **隐式 Intent：** 系统通过 `IntentFilter` 匹配合适的组件再启动

---

### 7. 启动服务的两种方式，除了生命周期还有什么区别？

- **`startService()`：** 启动的 Service 独立运行，需要手动停止，客户端无法直接调用 Service 方法；适合后台独立运行的任务
- **`bindService()`：** 启动的 Service 与客户端绑定，通过 `IBinder` 可以直接通信，当所有客户端解绑后 Service 会自动销毁；适合需要和组件交互的服务

---

### 8. 安卓主线程为什么不能做耗时操作，会出现什么问题，超过多久会出问题？

Android 的主线程（UI 线程）负责 UI 绘制、用户输入事件处理、组件生命周期回调。主线程内部通过 `Looper` + `MessageQueue` 按顺序处理消息，是单线程模型。

如果在主线程耗时操作，就会：

- 阻塞 MessageQueue
- 导致新的 UI 绘制和输入事件无法及时处理
- 界面无法刷新

因此会出现**界面卡顿、点击无响应**等问题。严重时触发 **ANR（Application Not Responding）**。

**有两个重要时间点：**

- **16ms：** 屏幕 60Hz 刷新率 → `1000ms / 60 ≈ 16.6ms`。如果 UI 绘制超过 16ms 未完成，就会掉帧、产生卡顿。
- **ANR 判定时间：**
  - Input 事件：5 秒
  - BroadcastReceiver：10 秒
  - Service：20 秒

---

### 9. 安卓启动模式的 SingleTask 和 SingleTop 的区别及各自使用场景？

**SingleTop：栈顶复用**

如果 Activity 已经在栈顶，不会重新创建，而是调用 `onNewIntent()`；如果不在栈顶，仍然会创建新实例。

- **场景：** 防止用户重复打开同一页面，比如详情页、聊天页。

**SingleTask：栈内唯一**

如果任务栈中已存在该 Activity，会复用实例并清除其上的 Activity，然后回调 `onNewIntent()`。

- **场景：** 应用主页、通知点击进入应用。

---

## 内存与 GC

### 1. 内存泄漏有哪些具体的例子，循环引用的具体例子？

**内存泄漏的本质是** 短生命周期对象持有了长生命周期对象。

常见的有：

- 静态对象持有了 Context
- 使用了非静态内部类 `Handler`

**循环引用示例：**

例如 Activity 持有 Manager，而 Manager 又通过 Callback 持有 Activity，这就形成循环引用。如果 Manager 是静态对象，就可能导致 Activity 内存泄漏。

---

### 2. Android 的 ART 的 GC 和 JVM 的 GC 区别是？

**首先是运行环境不同：**

JVM 主要运行在服务器环境，更关注吞吐量；而 ART 运行在移动设备上，更关注低卡顿和低功耗，所以会尽量减少 STW 时间。

**第二是 GC 结构不同：**

- JVM 采用比较典型的分代 GC，新生代和老年代结构明显，常见收集器有 CMS、G1 等；
- ART 并不是严格的分代模型，主要有 Sticky GC、Partial GC、Full GC，通过缩小回收范围来减少 GC 停顿。

**第三是优化目标不同：**

- JVM 的 GC 更关注整体吞吐量，虽然像 CMS、G1 也有并发阶段，但仍然可能出现相对较长的 STW；
- ART 的 GC 更强调并发回收，尽量把 STW 控制在很短时间，避免 Android 出现 UI 卡顿。

> **总结：** JVM 的 GC 更偏向服务器吞吐量优化，而 ART 的 GC 更偏向移动设备的低暂停时间优化。

---

### 3. Sticky GC、Partial GC、Full GC 的作用域？

ART 的 GC 主要基于 CMS（Concurrent Mark Sweep）思想实现：

**Sticky GC：**

- 作用域最小
- 只扫描最近分配的对象（allocation stack / young objects）
- 回收速度最快
- 类似 JVM 的 Minor GC

**Partial GC：**

- 作用域中等
- 回收整个应用堆的一部分 Region / Space
- 比 Sticky GC 扫描范围更大

**Full GC：**

- 作用域最大
- 扫描整个堆
- 开销最大，一般在内存压力较大时触发

---

### 4. LeakCanary 的原理？

LeakCanary 通过监听 Activity/Fragment 的 `onDestroy()` 找到理论上应该被回收的对象，再通过 `WeakReference` + `ReferenceQueue` 判断对象是否真的被 GC 回收，如果检测到没有回收，就会自动 dump heap 导出堆信息并提示可能的泄漏对象。

**具体流程：**

**第一步：监听 Activity / Fragment 的生命周期**

LeakCanary 会通过 `Application.registerActivityLifecycleCallbacks` 监听 Activity 生命周期。监听到 Activity 或 Fragment 执行 `onDestroy()` 时，说明这个对象理论上应该可以被回收了。

**第二步：使用 WeakReference 监控对象**

LeakCanary 会把这个 Activity 包装成 `WeakReference`，并放进一个监控集合。因为 `WeakReference` 不会阻止 GC 回收对象。如果这个 Activity 没有被其他对象引用，在下一次 GC 时就会被回收。

**第三步：通过 ReferenceQueue 判断是否被回收**

LeakCanary 会配合 `ReferenceQueue`：

- 如果对象被 GC 回收，对应的 `WeakReference` 会进入 `ReferenceQueue`

所以如果 `onDestroy()` 一段时间后这个引用没有进入队列，说明：**这个对象仍然被其他对象持有引用，可能发生了内存泄漏。**

---

### 5. 弱引用的底层原理？和虚引用的区别是？

在 JVM 可达性分析过程中，`WeakReference` 的 `referent` 不会被当作强可达路径参与对象图遍历；当对象只被弱引用关联时，会被判定为**弱可达对象**，并在 GC 的处理阶段被回收，同时清空 `WeakReference` 的 `referent`，如果绑定了 `ReferenceQueue`，则会将该引用入队。

| 引用类型 | 说明 |
|---------|------|
| **WeakReference** | 对象在 GC 判定为弱可达时，JVM 会先清空 `referent`，然后再入队。此时对象已经没有可访问引用，基本等价于"对象已经被清理/即将被回收"的通知 |
| **PhantomReference** | 对象在**即将被真正回收前**会被入队，此时 JVM 保证对象不会再被访问或复活，主要给程序一个机会去做资源清理（比如释放堆外内存） |

> 所以可以理解：
> - **WeakReference：** 对象被判定可回收后的通知
> - **PhantomReference：** 对象真正回收前的最后通知

---

### 6. 怎么排查内存泄漏问题？

**一般分三步：**

**第一步：通过内存监控发现问题**

怀疑某个功能有内存问题，先通过 `adb` 查看进程内存使用情况：

```bash
adb shell dumpsys meminfo <包名>
```

通过 `meminfo` 里的 PSS、Java Heap、Native Heap 来判断是 Java 层泄漏还是 Native 层问题。

反复操作某个场景（比如反复进入退出某个 Activity 或页面），如果发现：

- Java Heap 持续增长
- 多次 GC 后内存仍然没有下降

说明可能存在对象没有被正确释放。

**第二步：使用 Heap Dump 或 LeakCanary 定位泄漏对象**

确认可能存在泄漏后，可以导出堆信息。如果是开发过程中，可以接入 LeakCanary 辅助定位泄漏。

**第三步：通过引用链分析 GC Root 找到具体原因**

---

### 7. AlarmManager、WorkManager、JobScheduler 的区别和联系？

**AlarmManager：** 早期 Android 中常用的定时任务机制，可以在指定时间或周期触发一个 PendingIntent（例如启动组件或发送广播）。优点是定时比较精确、使用简单。但缺点是容易被应用频繁使用唤醒设备，比较耗电，而且在 Doze 等省电机制下很多任务会被系统延迟执行。

**JobScheduler：** 后来 Android 引入的系统级后台任务调度，开发者可以为任务设置执行条件（比如需要网络、设备充电或者设备空闲等）。启动一个 `JobService`，条件满足后回调的方式执行任务，从而减少设备唤醒次数，更加省电。但只支持 Android 5.0 及以上版本。

**WorkManager：** 为了解决不同 Android 版本兼容的问题，Google 在 Jetpack 中封装的框架。它会根据系统版本自动选择底层实现：**高版本系统上使用 JobScheduler，低版本上使用 AlarmManager**。同时还支持任务链、任务重试以及任务持久化等功能。

> **总结：**
> - **AlarmManager：** 主要用于定时任务，时间较精确
> - **JobScheduler：** 系统级后台任务调度，可以设置执行条件
> - **WorkManager：** Jetpack 提供的统一后台任务方案，内部适配 JobScheduler 和 AlarmManager，现在官方更推荐使用 WorkManager 来执行后台任务。

---

## 网络

### 1. TCP 的复用是怎么做的？

### 2. HTTP 1.1 和 HTTP 2.0 连接复用的区别？

### 3. OkHttp 的连接池是什么，怎么用的？

### 4. 电量和网络对 App 的影响有了解过吗？

电量和网络属于系统资源约束，Android 会通过电源管理和网络策略限制 App 的后台行为。

**电量方面：** 系统有 Doze 和 App Standby 等省电机制。当设备长时间不用或者 App 很少被打开时，系统会限制后台任务、网络访问以及定时任务的执行，所以很多后台任务需要通过 WorkManager 或 JobScheduler 来调度。

**网络方面：** 主要是网络状态和网络类型的变化，比如 WiFi、移动网络或者无网情况。App 需要监听网络变化，并做好重试、缓存或者在合适的网络条件下再执行请求。

> **总结：** App 需要根据电量状态和网络状态合理调度任务和网络请求，避免在系统受限的情况下执行大量后台操作。

---

## Handler 与线程

### 1. Handler 的 `postDelay` 为什么切到后台就不生效了？

Handler 的 `postDelayed` 本质是把 Runnable 封装成 Message，并计算执行时间 `when = SystemClock.uptimeMillis() + delay`，然后按时间顺序插入 MessageQueue。

Looper 不断调用 `MessageQueue.next()`，当 `当前 uptimeMillis >= message.when` 时就取出执行。

由于 `postDelayed` 使用的是 `SystemClock.uptimeMillis()` 作为时间基准，而 `uptimeMillis` 在设备 deep sleep 时不会增长。当应用进入后台、设备进入 sleep 时计时会暂停，因此任务会被明显延迟，看起来像 `postDelayed` 失效。

> 因此：Handler 只适合进程存活期间的短期调度。如果需要在后台或设备休眠期间仍然按时执行，通常需要使用 AlarmManager 或 WorkManager。

---

## View 与 UI

### 3. 安卓中动画有哪些种类？

Android 主要有 **三种动画**：

**1. View Animation（补间动画 / Tween Animation）**

对 View 做简单平移缩放等变换。但**只改变显示效果，不会改变 View 的真实属性和位置**——比如点击新位置不会响应，View 实际还是原来的位置。

**2. Frame Animation（帧动画）**

一张一张图片连续播放形成动画。缺点是内存占用大，而且动画效果固定、不够灵活。

**3. Property Animation（属性动画）**

Android 3.0 引入，现在开发**最常用**。直接修改对象属性值，可以作用于**任何对象**，View 的状态会真实改变。

---

### 4. 悬浮窗怎么实现？

悬浮窗本质是通过 WindowManager.addView() 创建一个新的系统 Window。应用提供 View 和 LayoutParams，系统通过 WMS 创建对应的 WindowState 并分配 Surface，从而显示在屏幕上。

应用创建 View 后，配置 WindowManager.LayoutParams（如 type、flags、位置等），

然后调用 addView() 添加到 WindowManager，

最终由 WindowManagerService 管理并显示到屏幕。

如果是跨应用显示的悬浮窗，一般需要使用 TYPE_APPLICATION_OVERLAY 类型，并申请 SYSTEM_ALERT_WINDOW 权限

### 5. 使用 `WindowManager.addView` 涉及哪些操作，假设两个窗口的层级一样会发生什么？
Android 的界面是通过 Window 作为显示单元。系统中可能同时存在很多窗口，比如：
- Activity 界面
- Dialog
- 输入法
- 状态栏
- 悬浮窗
这些窗口需要统一管理它们的 显示层级、输入分发和显示合成，
因此 Android 在系统进程里有一个 WindowManagerService（WMS） 专门负责管理所有 Window。
本质是通过 WindowManager.addView() 将这个 View 以及它对应的 Window 参数注册到系统的 WindowManagerService，从而在系统中创建一个可管理的 Window

但是应用进程和系统进程是隔离的，系统只负责管理 Window 和 Surface，并不会直接管理应用里的 View 树。

因此在应用进程里通过 ViewRootImpl 来：
- 维护 View 树，负责 View 的 measure / layout / draw
- 负责和 WindowManagerService 建立通信

在调用 WindowManager.addView() 时，系统会做这样几件事情：
1. 首先在应用进程中创建 ViewRootImpl。
它会把 View 树和 Window 绑定起来，也就是说，一个 Window 对应一棵 View 树，而 ViewRootImpl 就是这棵树的根控制者。
2. ViewRootImpl 会通过 Binder 调用 WindowManagerService，请求系统创建一个新的 Window。
3. WMS 收到请求后会创建一个 WindowState 对象来表示这个窗口，并把它加入系统的 Window 列表。
4. 当窗口加入系统之后，WMS 会为这个窗口分配一个 Surface。
5. Surface 是一个可以被系统合成到屏幕上的图层，最终这些图层会交给 SurfaceFlinger 统一进行合成并显示到屏幕上。

### 假设两个窗口的层级一样会发生什么？
系统中会同时存在很多 Window，因此需要一个 Z 轴层级体系。
Android 使用 Window type 来定义窗口的大致层级，例如：
- 状态栏
- 输入法
- 应用窗口
- 壁纸窗口
不同 type 的窗口会被放在不同的层级，从而保证关键系统 UI 不会被普通应用覆盖。
但是 type 只能决定大层级。

如果两个 Window 的 type 相同，它们处在同一个层级，这时候系统还需要一个规则决定 谁在上面谁在下面。
Android 的规则是：按照窗口加入系统的先后顺序排序。
因此结果就是：
- 先添加的 Window 在下面
- 后添加的 Window 在上面


### 6. 三指、四指操作怎么实现？
App 级多指操作
因为 Android 的触摸事件最终会分发到 View 层，并通过 MotionEvent 表示多个触点信息，可以通过 getPointerCount() 获取手指数，通过 pointerId 跟踪每个手指的运动轨迹。
所以 在 View 或 ViewGroup 的 onTouchEvent() / onInterceptTouchEvent() 中，可以根据 pointerCount 判断是否为三指或四指触摸，并结合 ACTION_POINTER_DOWN、ACTION_MOVE 等事件以及触点移动方向进行手势识别。
因此 当检测到三指或四指并且运动轨迹满足条件时，就可以在应用内触发对应功能，例如图片缩放、多指滑动操作等。

---
系统级多指操作
因为 系统级手势需要在任何应用界面都能生效，并且不能被应用拦截。
所以 Android 会在 Input 事件分发阶段（如 InputReader / InputDispatcher 或系统手势模块）统一监听多点触控事件，在事件分发到应用之前进行手势识别。
因此 当系统检测到特定的三指或四指手势（例如三指下滑）时，就会直接触发系统功能，例如截图、返回桌面或切换应用，而不会把该手势继续分发给应用处理。




---

## 架构

### 1. MVP 和 MVVM 的区别？

---

## 项目与工程

### 1. 如果需要执行一些耗时的操作，你会怎么做？

耗时操作需要放到子线程执行，避免阻塞主线程导致 ANR，任务完成后再切换到主线程更新 UI。

在实际开发中一般会使用线程池、Handler 或一些框架（例如协程、RxJava）来管理异步任务，避免频繁创建线程带来的开销。

---

### 2. 安卓打出的包有哪些文件？

Android 打包生成的是 APK 文件，本质是一个 zip 压缩包，里面主要包含：

- **AndroidManifest.xml** — 声明组件和权限
- **classes.dex** — 代码编译后的字节码
- **res / assets / resources.arsc** — 资源目录、资源索引（R.id 查询）
- **lib** — 动态库
- **META-INF** — 签名信息等

---

### 3. 怎么用 AI 工具开发，需要注意什么？

### 4. 了解 Skills 和 MCP 吗？

---

> **备注：** 社招除了头部大厂，几乎不会考算法。八股文偏多，但其实即使基本能回答出来，作用也不大。面试官的重点在于**工作经历的匹配性**、**你的稳定性**，还有各种**软实力**。
