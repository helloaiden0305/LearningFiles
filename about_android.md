# Android 积累

---

## Android 打出的包有哪些文件？

Android 打包生成的是 APK 文件，本质是一个 zip 压缩包，里面主要包含：

1. AndroidManifest.xml 声明组件和权限
2. META-INF 里的 apk 签名信息等。
3. 代码编译成的 classes.dex
4. res 资源目录、assets 资源目录、resources.arsc、(各类资源目录、arsc 资源索引 R.id 查询)
5. lib.so 动态库

---

## ！电量和网络对 App 的影响?

电量和网络属于系统资源约束，Android 会通过电源管理和网络策略限制 App 的后台行为。

电量方面，系统有 Doze 和 App Standby 等省电机制。当设备长时间不用或者 App 很少被打开时，系统会限制后台任务、网络访问以及定时任务的执行，所以很多后台任务需要通过 WorkManager 或 JobScheduler 来调度。

网络方面，主要是网络状态和网络类型的变化，比如 WiFi、移动网络或者无网情况。App 需要监听网络变化，并做好重试、缓存或者在合适的网络条件下再执行请求。

**总结就是：** App 需要根据电量状态和网络状态合理调度任务和网络请求，避免在系统受限的情况下执行大量后台操作。

---

## 哪些性能优化工作？

启动优化、UI 卡顿优化和内存优化。

启动阶段通过 Trace 工具分析主线程耗时，拆分初始化任务并延迟加载；
卡顿方面通过 Systrace 分析掉帧原因，减少主线程耗时操作并优化 RecyclerView 和布局；
内存方面通过 LeakCanary 排查泄漏并优化对象引用关系。

---

### 提高 Android APP 安全性的常见手段

- 代码混淆  

使用 ProGuard/R8 混淆类名、方法名，降低反编译后的可读性。

- 敏感信息加密  

对 API Key、配置文件、系统参数做 AES/RSA 加密，避免明文暴露在 APK 中。

- 签名与完整性校验  

这是 Android 系统在 安装 APK 时 的安全机制：系统会读取 APK 内的证书，用公钥验证签名，确保文件没有被篡改。它的原理就是 密钥对的用法。

---

### 安卓 APP 签名中密钥对的用法

- 私钥（Private Key）：开发者持有，用来对 APK 文件摘要加密，生成数字签名。

- 公钥（Public Key）：打包时放进 APK 的证书里，安装时系统用它来解密签名。

1. 打包时

  - 对 APK 文件内容做哈希运算 → 得到文件摘要。

  - 用私钥加密摘要 → 得到数字签名。

  - 把数字签名和公钥证书一起打包进 APK（CERT.RSA）。

2. 安装时

  - 系统读取 APK 内的公钥证书。

  - 用公钥解密数字签名，得到开发者当初加密过的摘要。

  - 系统再自己对 APK 文件重新计算一次摘要。

  - 比对两个摘要：一致 → 文件没被篡改，签名有效；不一致 → 安装失败。

3. 版本升级校验

  - 手机上已安装的 APK 和你要更新的 APK。升级时系统会比对两者的签名证书，如果公钥一致说明是同一个私钥签发的，就允许覆盖安装；否则安装失败。”


---

## 四大组件相关

### Activity 生命周期冷启动顺序？

`onCreate → onStart → onResume`

触发原因（辅助记忆）：

- **onCreate：** Activity 第一次创建，加载布局、初始化 View 树（savedInstanceState = null）
- **onStart：** Activity 已经可见，对应的 Window 已经添加到 WindowManager，但还不是焦点窗口，暂时不能接收用户输入事件。
- **onResume：** Activity 的 Window 成为焦点窗口（focused window），开始接收用户输入事件；InputDispatcher 会把输入事件派发到这个 Window，用户可以与界面交互。

**一句话：** 创建 → 可见 → 可交互。

---

### onStart 和 onResume 的区别？

**标准答案：**

- **onStart：** 可见但不可交互（Window attach，但未获得输入焦点）
- **onResume：** 可见且可交互（InputDispatcher 切换输入通道）

**触发原因：**

系统把 Activity 放到前台需要两步：

1. 让它"显示"出来（onStart）
2. 让它"接收事件"（onResume）

---

### onPause 和 onStop 的区别？

**标准答案：**

- **onPause：** 失去焦点但仍可见（比如弹出对话框）
- **onStop：** 完全不可见（被新 Activity 完全覆盖）

**触发原因：**

系统判断 Activity 是否"可见"决定是否触发 onStop。

---

### 从 A 跳到 B 的生命周期顺序？

从 A → B 的跳转属于"前台切换"，不是"配置变化"也不是"进程回收"。所以 A 不会被销毁。只会走：onPause → onStop。

```
A.onPause → B.onCreate → B.onStart → B.onResume → A.onStop
```

**触发原因：**

- A 必须先 onPause（让出焦点）
- B 才能开始创建
- 当 B 完全显示后，A 才会 onStop

**关键点：** onPause 必须轻量，否则 B 启动卡顿。

---

### ！按返回键从 B 回到 A 的生命周期顺序？

返回键的语义 = 从 Task 栈中移除当前 Activity。被移除的 Activity 必须 onDestroy，否则它会一直留在栈顶，永远无法退出。

```
B.onPause → A.onRestart → A.onStart → A.onResume → B.onStop → B.onDestroy
```

**触发原因：**

Android 为了保证界面连续性，必须：先让 A 恢复前台，再让 B 完全退出。

- A 仍在 Task 栈中 → 触发 onRestart
- B 被移除 → onStop → onDestroy

---

### ！横竖屏切换时生命周期顺序（未声明 configChanges）？

**标准答案：**

```
onPause → onStop → onSaveInstanceState → onDestroy → onCreate（带 Bundle 重建） → onStart → onResume
```

**触发原因：**

- 旋转屏幕触发 Configuration Change
- 系统必须重建 Activity
- savedInstanceState 用于恢复 UI 状态

**注意：** 折叠屏展开/合拢同理。

---

### ！哪些 Configuration Change 不走 onRestart？为什么？

**标准答案：** 不走 onRestart 的情况比如：

- 深色模式切换
- 折叠屏展开/合拢

**原因：**

只有"触发 Activity 重建"的 Configuration Change 才不走 onRestart。

```
onPause → onStop → onDestroy → onCreate
```

---

### *）为什么国内 ROM 会出现"秒级 onStop/onDestroy"？

因为厂商为了省电/省内存，会对后台 Activity/进程做激进管控：

- Activity 一退到后台，很快就被 finish 掉
- 所以从 onStop 到 onDestroy 的时间非常短，看起来像"秒级 onDestroy"
- 所以 onStop → onDestroy 很快触发

---

### *）为什么小米/华为权限弹窗只触发 onPause 不触发 onStop？

权限弹窗是一个**透明 Activity**，不会遮挡当前 Activity。因此底层 Activity 仍然"可见"，只会：

- onPause（失去焦点）
- 不会 onStop（仍可见）

---

### ！微信/支付宝为什么会出现奇怪的 Activity 栈顺序？

因为它们使用 singleTask / singleInstance / Task 复用策略。导致：

- Activity 不按常规方式入栈
- 返回栈顺序可能与预期不一致
- 某些 Activity 会复用已有 Task

**原因：** 为了加快启动速度、减少内存占用。

---

### ！为什么 onPause 不能做耗时？

因为**新 Activity 启动前必须等待旧 Activity 的 onPause 执行完**。如果 onPause 超过 10ms，会导致：

- 新 Activity 启动卡顿
- 用户感知"点击延迟"

**原因：** ActivityThread 的启动流程是串行的。

---

### ！后台进程被系统杀死后，回前台为什么直接走 onCreate？

因为进程被杀死回收后：

- Activity 实例已经不存在
- 系统只能重新创建 Activity
- 因此直接走 onCreate（带 savedInstanceState）

不会走 onRestart，因为 Activity 根本没存活。

---

### ！为什么 onSaveInstanceState 在 API 28+ 变成 onStop 后调用？

因为系统行为调整：

API 27 及之前：onSaveInstanceState() 会在 onPause() 之后调用。但这是 Activity 可能还在前台短暂可见，如果这时保存状态，可能导致 UI 状态丢失或不一致。

**为了避免 UI 状态丢失，Android 9+ 把 onSaveInstanceState 放到 onStop 之后，完全不可见后调用。**

**注意：** 不要在这里存大对象，只存 UI 状态。

---

### ！Fragment 的创建流程和生命周期？

**前提：** Fragment 不能独立存在，必须依附在 Activity 上，由 FragmentManager 管理。

**1. 创建并添加 Fragment 的典型流程：**

```java
FragmentManager fm = getSupportFragmentManager();
FragmentTransaction ft = fm.beginTransaction();
ft.add(R.id.container, new FragmentA(), "A");
ft.commit();
```

**对应的生命周期大致是：**

- onAttach() — Fragment 绑定到 Activity
- onCreate() — 初始化逻辑（非 UI）
- onCreateView() — 创建 Fragment 的 View
- onViewCreated() — View 已创建，做 UI 初始化
- onStart() — Fragment 对用户可见
- onResume() — Fragment 可交互

**一句话：** Fragment 的创建是通过 FragmentTransaction（add/replace）触发的，生命周期由系统按需回调。

---

### 从一个 Fragment 切换到另一个 Fragment？

#### 场景一：页面跳转式切换（类似新页面）

**做法：** 用 `replace()`

```java
getSupportFragmentManager()
    .beginTransaction()
    .replace(R.id.container, new FragmentB(), "B")
    .addToBackStack(null) // 可选：支持返回键回到上一个 Fragment
    .commit();
```

**特点：**

- `replace()` = 先 remove(old) 再 add(new)
- 旧 Fragment 的 View 会被销毁（onDestroyView()），如果没进回退栈，实例也可能被销毁
- 适合"真正的页面跳转"

---

#### 场景二：Tab/底部导航式切换（频繁切换）

**做法：** `add()` + `show()` / `hide()`

**初始化时：**

```java
FragmentTransaction ft = fm.beginTransaction();
ft.add(R.id.container, fragmentA, "A");
ft.add(R.id.container, fragmentB, "B");
ft.hide(fragmentB);
ft.commit();
```

**切换时：**

```java
FragmentTransaction ft = fm.beginTransaction();
ft.hide(fragmentA);
ft.show(fragmentB);
ft.commit();
```

**特点：**

- Fragment 实例常驻，View 也可以常驻
- 切换只是显示/隐藏，性能好
- 适合 Tab、底部导航、多页签场景

**总结：**

- **页面跳转式：** 用 `replace()`，销毁旧 Fragment 的视图，创建新 Fragment，适合真正的页面切换。
- **Tab 切换式：** 用 `add()` + `show()/hide()`，保留 Fragment 实例，只切换显示状态，适合频繁切换场景。

---

### 为什么用 add/hide/show 取代 replace？

replace 在执行时，实际上会把当前容器里的 Fragment remove 掉，然后再 add 一个新的 Fragment。
这样会导致原来的 Fragment 走一套完整的生命周期，比如 onPause → onStop → onDestroyView → onDestroy，
等再次切回来时又要重新创建 View 和重新初始化数据，开销比较大。

而使用 add + hide/show 的方式时：
第一次进入时 add Fragment
切换时只是 hide 当前 Fragment，show 目标 Fragment
这样 Fragment 实例和 View 都会被保留在内存中，不会反复创建和销毁，因此提升切换性能和流畅度，可以保留页面状态（滚动位置、输入内容等）
所以在像 Tab 切换、底部导航、多页面切换 这种场景下，一般都会用 add + hide/show 来代替 replace。

---

### 为什么官方推荐用 replace？

官方推荐 replace，是因为它会正确销毁旧 Fragment，使生命周期更清晰、内存管理更安全，也能避免 Fragment 重叠等问题；
而 add/hide/show 虽然切换性能更好，但需要开发者自己管理状态，容易出错，所以只适合少量固定页面的场景，比如底部 Tab。

---

### 采取 hide/show 后 Fragment 怎么通信？

如果使用 hide/show 管理 Fragment，由于 Fragment 实例一直存在，通信方式通常有：

- 通过 Activity 中转调用
- 使用 Activity 作用域 ViewModel 共享数据（官方推荐）

---

### 为什么 Fragment 在 hide 后 LiveData 有时不会更新，看起来像"卡住"？

因为 hide 并不会改变 Fragment 的生命周期状态，而 LiveData 是否分发数据是 根据 Lifecycle 状态判断的。
LiveData 的分发规则是：只要 Observer 的 Lifecycle ≥ STARTED，就会收到数据。

也就是：

- STARTED
- RESUMED

这两个状态都会接收 LiveData。

当你使用 add + hide/show 时，hide 只是把 Fragment 的 View 设为不可见，Fragment 仍然在 FragmentManager 中，并且生命周期通常仍然是：

- RESUMED 或 STARTED

所以 LiveData 仍然认为这个 Fragment 是 active observer，因此继续分发数据。
但 View 被隐藏后 UI 没有刷新，show 回来后"看起来卡住"。

---

### Fragment 懒加载是什么？

在 Fragment 真正对用户可见的时候才进行数据加载或耗时操作，而不是 Fragment 创建的时候。

常见于 ViewPager，几个页面来回切换。

通过生命周期或可见性控制数据加载，能够减少无效操作，提升页面响应的感受。

---

### Activity 与 Fragment 之间常见的几种通信方式？

#### 双向：通过 ViewModel + LiveData（推荐方式，解耦且生命周期安全）

> ViewModel + LiveData 的通信其实是通过 Activity 作为宿主来共享 ViewModel。Activity 创建 ViewModel，Fragment 拿同一个实例，这样就能共享数据，解耦又生命周期安全。

```java
// ViewModel
class SharedViewModel extends ViewModel {
    MutableLiveData<String> message = new MutableLiveData<>();
}

// Activity 中
SharedViewModel vm = new ViewModelProvider(this).get(SharedViewModel.class);
vm.message.observe(this, msg -> Log.d("收到: " + msg));
// Activity 主动更新数据
button.setOnClickListener(v -> vm.message.setValue("Activity 发消息"));

// Fragment 中
SharedViewModel vm = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
vm.message.observe(getViewLifecycleOwner(), msg -> Log.d("收到: " + msg));
// Fragment 主动更新数据
view.findViewById(R.id.btn).setOnClickListener(v -> vm.message.setValue("Fragment 发消息"));
```

```
[Activity] --------------------+
   |                           |
   | 1. 获取 SharedViewModel   |
   | 2. 观察 vm.message        |
   | 3. 点击按钮 -> setValue   |
   |                           |
   v                           v
SharedViewModel (持有 LiveData<String>)
   ^                           ^
   |                           |
   | 4. 数据更新 -> 通知观察者 |
   |                           |
[Fragment] --------------------+
   | 1. 获取同一个 SharedViewModel (requireActivity)
   | 2. 观察 vm.message
   | 3. 点击按钮 -> setValue
```

#### Activity → Fragment：通过 FragmentManager

Activity 找到 Fragment 实例，直接调用方法。

```java
// Activity 内部
MyFragment fragment = (MyFragment) getSupportFragmentManager()
        .findFragmentById(R.id.fragment_container);

if (fragment != null) {
    fragment.updateUI("Hello Fragment");
}
```

```java
public class MyFragment extends Fragment {
    TextView textView;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_layout, container, false);
        textView = view.findViewById(R.id.textView);
        return view;
    }

    // Activity 可以直接调用这个方法
    public void updateUI(String msg) {
        textView.setText(msg);
    }
}
```

#### 通过 Bundle

```java
Bundle args = new Bundle();
args.putString("key", "Hello Fragment");

MyFragment fragment = new MyFragment();
fragment.setArguments(args);

getSupportFragmentManager()
    .beginTransaction()
    .add(R.id.container, fragment)
    .commit();
```

**总结：**

- **双向：** 推荐用 ViewModel + LiveData（observe 监听），能实现双向通信，解耦且生命周期安全。LiveData 本质就是监听容器的数据变化，然后通知。生命周期安全体现在：容器销毁时自动解绑不会触发回调，避免内存泄漏和崩溃。
- **Activity → Fragment：** Activity 也可以通过 FragmentManager 调用 Fragment 方法，并且用 Bundle 在创建时传参。

---

### LaunchMode 的应用场景？

- **standard：** 默认，每次启动都新建实例。不用声明的默认设置
- **singleTop：** 栈顶已有则复用，常用于通知栏点击。
- **singleTask：** 整个任务栈只有一个实例，常用于首页。
- **singleInstance：** 独占任务栈，常用于电话、闹钟等全局入口。不能和其他 Activity 混在一起，防止干扰。

---

### SingleTask 和 SingleTop 的区别及使用场景？

**SingleTop：栈顶复用**

如果 Activity 已经在栈顶，不会重新创建，而是调用 `onNewIntent()`；如果不在栈顶，仍然会创建新实例。

- **场景：** 防止用户重复打开同一页面，比如详情页、聊天页。

**SingleTask：栈内唯一**

如果任务栈中已存在该 Activity，会复用实例并清除其上的 Activity，然后回调 `onNewIntent()`。

- **场景：** 应用主页、通知点击进入应用。

---

### ！对于 Context，怎么理解？

- **ApplicationContext：** 进程级别，全局环境，生命周期长。
- **ActivityContext：** 界面级别，和 Activity 生命周期绑定。
- **区别：** 前者适合资源访问、系统服务；后者适合 UI 操作。
- **注意：** 避免内存泄漏，不要把 ActivityContext 长期持有。

**答版：**

> Context 是应用和系统交互的入口。常见有 ApplicationContext（进程级，提供一个全局环境，适合资源访问和系统服务）和 ActivityContext（每个界面独立一个，和 Activity 生命周期绑定，适合 UI 操作）。他们本质上都是内含了一个 contextimpl。默认不声明就是 ActivityContext。注意不要长期持有 ActivityContext，避免内存泄漏。

---

### IntentFilter 是什么？有哪些使用场景？

**定义：** 在 manifest 里用于声明组件能响应哪些 Intent（action、category、data）。

**例如：**

```xml
<activity android:name=".MyActivity">
    <intent-filter>
        <!-- 响应系统的 VIEW 动作，比如打开网页 -->
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <!-- 指定数据类型：这里是 http/https 链接 -->
        <data android:scheme="http" />
        <data android:scheme="https" />
    </intent-filter>
</activity>
```

**场景：**

- Activity 响应特定 action（如 ACTION_VIEW）。
- BroadcastReceiver 监听系统广播。
- Service 响应特定启动请求。

---

### 显式 Intent 和隐式 Intent 的区别？

**指定方式不同：**

- **显示 Intent：** 明确指定要启动的组件类名（ComponentName）
- **隐式 Intent：** 不指定具体组件，只描述动作（action）和数据（data），系统匹配

**使用场景不同：**

- **显示 Intent：** 一般用于应用内部组件之间的跳转（如 Activity → Activity）
- **隐式 Intent：** 用于调用系统组件或其他应用功能（如打开浏览器、分享、拨号）

**系统处理方式不同：**

- **显示 Intent：** 系统直接找到目标组件并启动
- **隐式 Intent：** 系统通过 IntentFilter 匹配合适的组件再启动

---

### 启动服务的两种方式的区别，生命周期以及使用场景？

- **startService：** 不会自动停止，调用一次就启动，直到 stopService() 或自己 stopSelf()。适合后台长期任务。
- **bindService：** 调用者和 Service 建立绑定，会返回一个用于交互的 binder 对象，调用者退出时自动解绑，适合需要与 Service 交互的场景（如音乐播放）。
- **生命周期：**
  - **start：** `onCreate() → onStartCommand() → （运行中） → onDestroy()`
  - **bind：** `onCreate() → onBind() → （交互中） → onUnbind() → onDestroy()`

---

### Service 如何进行保活？

**强调：** Android 后续版本对保活限制越来越严，Android 不让保活，是因为后台进程会耗电、占资源、影响用户体验，还可能被滥用。所以系统不断加强后台限制，鼓励使用前台服务和 WorkManager 这种"可控、可见、可管理"的方式。

- **提升优先级：** 前台服务（startForeground）。让系统认为任务对用户可见，优先级高，不容易被杀。
- **JobScheduler / WorkManager** 定时拉起。

---

### 现代 Android 中如何优雅地处理后台任务？

**最佳实践是：** 根据任务类型选择合适的 API。

1. 短时并发用协程/线程。
2. 延迟或条件执行用 WorkManager。
3. 持续运行且用户必须知晓的任务用前台服务。

**WorkManager 是大多数场景的推荐方案**，因为它能保证任务在应用退出或设备重启后仍能执行。

---

### ！WorkManager 和 Service 的区别与使用场景？

**核心区别：**

> WorkManager 和 Service 都能处理后台任务，但定位不同：
> - Service 是 Android 最基础的后台组件，用来执行长时间运行的操作，可以在前台或后台运行。
> - WorkManager 是 Jetpack 提供的任务调度框架，专注于一次性或周期性任务，并能在应用退出或设备重启后继续执行。

- **长时、用户必须知晓的任务 → Service（尤其是前台 Service）。**
- **一次性或周期性、需要可靠执行的任务 → WorkManager。**

WorkManager 是现代 Android 推荐的后台任务方案，Service 只在必须持续运行时使用。

---

### ！应用之间分享数据怎么实现？

- **提供方：** 通过 ContentProvider 封装数据（通常是 SQLite），并在 Manifest 里声明。
- **调用方：** 通过 ContentResolver + URI 访问数据，就像访问数据库一样。
- **意义：** 这是 Android 官方提供的跨应用数据共享机制，带有权限控制和统一接口。

**一句话答：**

> 跨应用访问是通过 ContentProvider 暴露数据，调用方用 ContentResolver + URI 来操作，实现跨进程共享。

---

### ！Intent 传输数据的大小有限制吗？如何解决？

**回答：**
Intent 传输数据的大小确实有限制，本质原因是 Intent 在系统内部通过 Binder IPC 传递，而 Binder 单次事务大小限制大约是 1MB。
如果传输数据过大会抛 TransactionTooLargeException。通常的解决方案是只通过 Intent 传递轻量数据，例如 ID 或路径，大数据通过文件、数据库或缓存等方式共享。

- **有限制**，大约 1MB 左右，超过会抛 `TransactionTooLargeException`。
- **解决方法：**
  - 持久化，用文件或数据库存储，再传路径或 ID。
  - 用 ContentProvider 或共享存储。
  - 用全局单例或缓存。

---

### BroadcastReceiver 与 LocalBroadcastReceiver 有什么区别？

**最大的区别在于通信机制和底层机制：**

- **全局广播：** 是跨进程的，底层依赖于 Binder，主要用于 APP 与系统、或者其他 APP 间的通信
- **本地广播：** 是应用内通信的，底层基于 Handler 实现的。

**现代已经废弃了 Local**，因为他的设计不够优雅和内存泄漏。现在的 APP 内部组件通信，通常会优先使用 LiveData、Kotlin 的 Flow 等。

---

### ！如何理解 Window 和 WindowManager？

**Window** 表示一个窗口，是界面显示的载体。

**定义：** 抽象类，代表一个窗口，承载界面显示。**实例化：** 应用层通常用的是 PhoneWindow（Window 的具体实现）。

在 Android 里我们看到的界面，比如 Activity 页面、Dialog、PopupWindow 本质上都对应一个 Window。

但平时开发不会直接操作 Window，而是操作一个 View，然后 View 会被放进 Window 里面。

```
Activity
  👇
Window
  👇
DecorView
  👇
View
```

其中，DecorView 是 Window 的顶层 View，常用的 `setContentView()`，最终就是把布局放到 DecorView 里面。

**WindowManager** 就是用来对 Window 进行操作的，添加、删除、更新等的一个客户端接口。应用通过调用 WindowManager，借由 Binder 调用 WMS，由 WMS 统一进行窗口管理、事件分发等。WMS 是 System Server 的系统服务之一，他才是真正负责窗口管理的位置。

---

### SQLite 和 SharedPreferences 的区别？

> **它们都是 Android 系统内置的存储方式。**
> SharedPreferences 底层是 XML 文件，适合保存轻量配置；缺点：主线程阻塞可能触发 ANR
> SQLite 是系统自带的数据库引擎，适合结构化数据和复杂查询。

平时用 SharedPreferences 保存用户设置，用 SQLite 存数据表。实践经验。

**简洁回答：**

- **SQLite：** 轻量级关系型数据库，适合结构化数据存储。
- **SharedPreferences：** 存储简单的键值对，适合轻量配置。

---

### 销毁一个 Activity，但是你还需要保存一些数据，你会怎么做？

**简洁回答：**

可以使用 `onSaveInstanceState()` 保存临时数据存到一个 Bundle 对象，通常在内存里，必要时序列化到磁盘，或持久化到 SharedPreferences / 数据库 / 文件。

**扩展理解：**

- **临时数据：** Bundle 保存 UI 状态。
- **长期数据：** SharedPreferences 或 SQLite。

---

### ！Android 的事件分发机制？滑动冲突是什么？怎么解决？

**事件分发，** 一般讨论事件的分发和流转。

从底层的系统视角来看，一个触摸事件是从硬件——系统——应用的完整过程：

1. **硬件与内核：** 触摸屏幕——屏幕硬件产生中断信号，Linux 的内核驱动把触摸信息记录到 `/dev/input` 设备节点
2. **系统服务层：** SystemServer 中的 IMS 专门负责处理输入事件，有两个核心线程：
   - **InputReader：** 循环从内核中读取事件
   - **InputDispatcher：** 把读到的事件，寻找处于焦点的 Window
3. **跨进程通信：** SystemServer → 跨进程 → APP 进程，传递点击事件
4. **应用层接收：** APP 进程中的 ViewRootImpl → 交给 DecorView → 交给 Activity

接着就是 **Activity → Window → DecorView → ViewGroup → View** 的逐层分发。

**分发过程中，主要涉及三个方法：**

- **dispatchTouchEvent** — 分发
- **onInterceptTouchEvent** — 拦截
- **onTouchEvent** — 消费

整个流程为 U 字形：

先从 Activity → Window → DecorView → ViewGroup → View 的逐层分发，确保准确消费。如果底层不处理，再从 View → ViewGroup → DecorView → Window → Activity 上抛，确保有兜底逻辑。

**滑动冲突：** 父 View 和子 View 都想处理同一个滑动事件，因此事件处理需要添加一个判定的标准，防止冲突。例如：

1. **外层拦截：** 重写父容器的 `onInterceptTouchEvent`，添加一个判定为上下滑的拦截。其余放给子 View。
2. **内层拦截：** 子 View 调用 `requestDisallowInterceptTouchEvent(true)`，通知父容器该事件他来处理。
---

###安卓广播中的标准广播和有序广播？

标准广播 (Normal Broadcast) sendBroadcast(intent);  

- 并发发送：所有接收者几乎同时收到广播。

- 无序性：接收顺序不确定，不能互相影响。

- 效率高：适合通知类场景，比如网络变化、电量变化。


有序广播 (Ordered Broadcast) sendOrderedBroadcast(intent, null)

- 顺序传递：接收者一个接一个按优先级执行。

- 可中断：前面的接收者可以拦截或修改广播内容，甚至阻止继续传递。

- 场景：比如短信接收，系统优先级高的接收者先处理，应用后处理。





---

## View 和 UI

### 屏幕视图刷新原理，涉及哪些进程、组件？

1. **硬件层：** 显示控制器发出 VSync 信号
   - 屏幕每隔一段时间（比如 16.6ms 对应 60Hz）发出一次 VSync。
   - 这个信号告诉系统：可以开始准备下一帧了。

2. **系统服务层：** SurfaceFlinger 响应 VSync
   - SurfaceFlinger 收到 VSync 后，开始合成所有应用的 Surface buffer。
   - 从每个应用的 Surface buffer 中取出最新的帧，根据 Z-order、透明度、动画等规则合成。

3. **应用层：** Choreographer 驱动 UI Thread
   - VSync 信号也会通过 Choreographer 通知应用层。
   - UI Thread 在下一次 VSync 到来时执行 `doFrame()` → 触发 `measure/layout/draw` → 把绘制结果写入 Surface。

4. **GPU/HAL 渲染**
   - SurfaceFlinger 合成后的帧交给 GPU 渲染。
   - HAL/驱动把渲染结果送到显示控制器。

5. **屏幕显示**
   - 显示控制器逐行扫描，把最终帧显示到屏幕。
   - 用户看到的就是这一帧画面。

```
硬件层 (Display Controller)         系统服务层 (SurfaceFlinger)         应用层 (UI Thread / Choreographer)
    |                                   |                                   |
    |--- VSync 信号 ------------------->|                                   |
    |                                   |--- VSync 信号 -----------------------------------------------> DisplayEventReceiver
    |                                   |                                   |--- Choreographer.doFrame()
    |                                   |                                   |    - measure/layout/draw
    |                                   |                                   |    - RenderThread + GPU 写入 Surface buffer
    |                                   |<----------------------------------|
    |                                   |--- 收集各 Layer 的 Surface buffer |
    |                                   |--- 合成帧 (Z-order/透明度/动画) --> GPU/HAL
    |                                   |                                   |
    |<---------------------------------- GPU/HAL 渲染结果 ------------------|
    |--- 显示控制器逐行扫描 ------------>|                                   |
    |                                   |                                   |
    |=== 用户看到最终画面帧 ============|                                   |
```

---

### ！View 的绘制流程？

1. Choreographer 接收 VSync 信号（来自 DisplayEventReceiver），调用 `doFrame()`，驱动 UI Thread 开始绘制。
2. ViewRootImpl（每个 Window 一个）负责发起 View 树的绘制流程，并生成 DisplayList。
3. **三大阶段：**
   - **measure：** 调用每个 View 的 `onMeasure()`，决定大小。
   - **layout：** 调用每个 ViewGroup 的 `onLayout()`，决定位置。
   - **draw：** 调用每个 View 的 `onDraw()`，生成绘制指令。
4. RenderThread + GPU
   RenderThread 首先向该 Window 对应的 BufferQueue 申请（dequeueBuffer）一块空闲的 Graphic Buffer（这里就是三缓冲发挥作用，提供空闲 Buffer 的地方）。
   GPU 根据 DisplayList 渲染到 Window 的 Surface buffer。
   渲染完成后，RenderThread 将这块填满画面的 Buffer 提交（queueBuffer）回 BufferQueue。
5. SurfaceFlinger 在下一次 VSync 合成所有应用的 Surface buffer，GPU 输出最终帧到屏幕。

---

### 3. 如何实现自定义 View 或自定义 ViewGroup？onMeasure 中的 MeasureSpec 是由什么决定的？

**核心步骤在于重写三个关键方法：**

1. **onMeasure：** 负责测量大小。这里的核心是 MeasureSpec，它是一个 32 位的 int 值，包含了测量模式（mode）和尺寸大小（size）。子 View 的 MeasureSpec 是由父容器的 MeasureSpec 和子 View 自身的 `LayoutParams` 共同决定的。
2. **onLayout：** 负责摆放位置。只有 ViewGroup 才需要，因为单独一个 View 是没有子 View 的。只有在自定义 ViewGroup 的时候，根据测量好的大小，安排子 View 的具体坐标。
3. **onDraw：** 负责绘制内容。利用 `canvas` 和 `Paint` 绘制出最终的视觉效果。

---

### RecyclerView 相对于 ListView 的优点？

**简洁回答：**

共同点是：都用来展示一组数据的列表。不同点是：

- **ListView：** 固定只能竖直列表。

**RecyclerView：** 更灵活，性能更好。

- RecyclerView 可以通过 LayoutManager 灵活控制布局（线性、网格、瀑布流）。
- RecyclerView 强制使用 ViewHolder。

> RecyclerView 强制使用 ViewHolder，相当于把控件引用提前缓存起来，避免频繁调用 `findViewById()`。这样在列表快速滚动时，性能更好，不卡顿。

---

### RecyclerView 为什么要用缓存池？为什么限制最大值？

RecycledViewPool（缓存池） 会把回收的 ViewHolder 缓存起来，当新的 item 需要时可以 **直接复用已有 ViewHolder**，避免再次 inflate，从而提升性能。

如果没有缓存池，当超过限制的 ViewHolder 被回收后，再次需要时仍然要 **重新 inflate View**，成本比较高。

限制最大值是限制缓存池中存在的 ViewHolder 数量，防止内存压力。

---

### 安卓中动画有哪些种类？

Android 主要有 **三种动画**：

**1. View Animation（补间动画 / Tween Animation）**

对 View 做简单平移缩放等变换。但**只改变显示效果，不会改变 View 的真实属性和位置**。例如点击新位置不会响应，View 实际还是原来的位置。

**2. Frame Animation（帧动画）**

一张一张图片连续播放形成动画。缺点是内存占用大，而且动画效果固定、不够灵活。

**3. Property Animation（属性动画）**

Android 3.0 引入，现在开发**最常用**。直接修改对象属性值，可以作用于**任何对象**，View 的状态会真实改变。

---

### 悬浮窗怎么实现？

悬浮窗本质是通过 `WindowManager.addView()` 创建一个新的系统 Window。

1. 应用提供 View 和 `LayoutParams`。
2. 系统通过 WMS 创建对应的 WindowState 并分配 Surface，从而显示在屏幕上。

如果是跨应用显示的悬浮窗，一般需要使用 `TYPE_APPLICATION_OVERLAY` 类型，并申请 `SYSTEM_ALERT_WINDOW` 权限。

应用创建 View 后，配置 `WindowManager.LayoutParams`（如 type、flags、位置等），然后调用 `addView()` 添加到 WindowManager，最终由 WindowManagerService 管理并显示到屏幕。

---

### ！使用 WindowManager.addView 涉及哪些操作？

Android 的界面是通过 Window 作为显示单元。系统中可能同时存在很多窗口，比如：

- Activity 界面
- Dialog
- 输入法
- 状态栏
- 悬浮窗

这些窗口需要统一管理它们的**显示层级、输入分发和显示合成**，因此 Android 在系统进程里有一个 **WindowManagerService（WMS）** 专门负责管理所有 Window。

本质是通过 `WindowManager.addView()` 将这个 View 以及它对应的 Window 参数注册到系统的 WindowManagerService，从而在系统中创建一个可管理的 Window。

但是应用进程和系统进程是隔离的，系统只负责管理 Window 和 Surface，并不会直接管理应用里的 View 树。

因此在应用进程里通过 **ViewRootImpl** 来：

- 维护 View 树，负责 View 的 measure / layout / draw
- 负责和 WindowManagerService 建立通信

在调用 `WindowManager.addView()` 时，系统会做这样几件事情：

1. **首先在应用进程中创建 ViewRootImpl。** 它会把 View 树和 Window 绑定起来，也就是说，一个 Window 对应一棵 View 树，而 ViewRootImpl 就是这棵树的根控制者。
2. **ViewRootImpl 会通过 Binder 调用 WindowManagerService**，请求系统创建一个新的 Window。
3. **WMS 收到请求后会创建一个 WindowState 对象**来表示这个窗口，并把它加入系统的 Window 列表。
4. **当窗口加入系统之后，WMS 会为这个窗口分配一个 Surface。**
5. **Surface 是一个可以被系统合成到屏幕上的图层**，最终这些图层会交给 SurfaceFlinger 统一进行合成并显示到屏幕上。

---

### ！假设两个窗口的层级一样会发生什么？

系统中会同时存在很多 Window，因此需要一个 Z 轴层级体系。

Android 使用 Window type 来定义窗口的大致层级，例如：

- 状态栏
- 输入法
- 应用窗口
- 壁纸窗口

不同 type 的窗口会被放在不同的层级，从而保证关键系统 UI 不会被普通应用覆盖。

但是 type 只能决定大层级。如果两个 Window 的 type 相同，它们处在同一个层级，这时候系统还需要一个规则决定**谁在上面谁在下面**。

Android 的规则是：**按照窗口加入系统的先后顺序排序。**

因此结果就是：

- **先添加的 Window 在下面**
- **后添加的 Window 在上面**

---

### 三指、四指操作怎么实现？

**App 级多指操作：**

- 触摸事件最终到达 View 层，由 MotionEvent 表示。
- 通过 `getPointerCount()` 判断手指数，通过 `pointerId` 跟踪每个手指。
- 在 `onTouchEvent` / `onInterceptTouchEvent` 中识别三指/四指手势。
- 只能在当前应用内部生效，例如图片缩放、多指滑动等。

### Surface 的三缓冲是什么？有什么作用？

对于每个 Window，各自管理一个 BufferQueue，应用需要绘制的时候从 Queue 申请一个 Buffer，SurfaceFlinger 从里面取出当前需要渲染的 Buffer 交给 GPU 渲染。

三缓冲（Triple Buffering）是 Android 4.1（Project Butter）引入的机制，它在原有的双缓冲（Front Buffer 和 Back Buffer）基础上，增加了一个额外的 Graphic Buffer（第三个缓冲池）。

它的核心作用是解决双缓冲机制下，复杂渲染场景导致掉帧的问题。

- 在双缓冲下，如果 GPU 渲染某一帧超时（应用侧，超过Vsync信号没绘制好当前帧，或者屏幕显示侧渲染某一帧超时，两种情况）
都会出现一个 Back Buffer 会被应用占用，另一个Front Buffer 被屏幕渲染占用。

此时没有多余的 Buffer 可以用来准备下一帧的数据，会出现掉帧卡顿。

- 引入三缓冲后，当应用侧RenderThread和屏幕显示把前两个 Buffer 都占用时，它可以立刻获取第三个 Buffer 开始计算下一帧。显著提升了画面的流畅度。

### 为什么不采用 4/5/6 缓冲？

- **1. 致命的输入延迟（操作”不跟手”）：** 增加 Buffer 意味着拉长了渲染流水线。如果采用 4 缓冲或 5 缓冲，用户的触摸事件（如滑动屏幕）被 CPU 处理后，生成的画面必须排队等待前面所有的 Buffer 轮转完毕才能显示到屏幕上。这会导致极差的交互延迟体验，用户会明显感觉屏幕反应慢半拍。
- **2. 巨大的内存消耗：** 每一个图形 Buffer 都会占用实打实的物理内存。以高分辨率屏幕为例，一张全屏画面可能需要几 MB 甚至十几 MB 的内存。如果全局 UI 系统都采用 4 缓冲或更多，会导致系统内存占用暴涨，进而引发频繁的 GC，甚至导致后台应用被大量清理。
- **3. 边际效益递减（无法解决根本的算力瓶颈）：** 三缓冲已经足以解决绝大多数的 CPU 和 GPU 互相等待问题。如果系统卡到了连 3 个 Buffer 都全部占满的地步，说明遇到了绝对的性能瓶颈（算力严重不足）。应该考虑别的角度优化业务了

---
###安卓有哪些常见的布局类型？

LinearLayout：子 View 按水平或垂直方向依次排列。

RelativeLayout：子 View 相对父容器或其他子 View 定位。

FrameLayout：子 View 堆叠显示，后添加的覆盖前面的。

ConstraintLayout：通过约束关系灵活控制位置和大小，适合复杂界面。

TableLayout：按行列组织子 View，类似表格。

GridLayout：网格形式排列，支持跨行跨列。




---

## 虚拟机和内存

### 安卓中的内存分为哪些部分，PSS 和 RSS 的区别，Bitmap 位于什么部分内存中？

Android 内存主要包括 **3 + 3**：

- Java Heap、Native Heap、Stack 属于进程私有内存
- Code/Dex、Shared Memory、Graphics 属于进程共享内存（通常通过 `mmap` 映射到进程地址空间，所以多个进程可以共享同一份物理内存）

**RSS 和 PSS** 都是衡量"某个进程"的内存占用指标：

- **RSS** 表示进程实际占用的物理内存，会把共享内存重复计算
- **PSS** ≈ `Private + (Code/Dex + Shared Memory + Graphics) / 进程数`

PSS 会按比例分摊共享内存，因此更能反映真实内存占用，Android 系统主要参考 PSS。

```
RSS ≈ Private + Code/Dex + Shared Memory + Graphics
```

比如共享的共 40MB、4 个进程使用：

- RSS：每个进程算 40MB
- PSS：每个进程算 10MB

在 Android 8.0 之后，**Bitmap 的像素数据存储在 Native Heap，而 Bitmap 对象在 Java Heap。**

---

### 内存泄漏有哪些具体的例子，循环引用的具体例子？

内存泄漏的本质是： 长生命周期对象持有了短生命周期对象。那么即使短周期对象已经销毁，GC 也认为它还在被长周期对象使用，就不会回收

常见的有：

- 静态对象持有了 Context
- 使用了非静态内部类 Handler
  
循环引用示例：

例如 Activity 持有 Manager，而 Manager 又通过 Callback 持有 Activity，这就形成循环引用。如果 Manager 是静态对象，就可能导致 Activity 内存泄漏。

---

### 为什么 Handler 会引起内存泄漏？

如果 Handler 是一个 Activity 中的非静态的内部类。Java 中的内部类是允许访问外部的内容的
那么在编译的时候，Java 编译器会给这个 Handler 创建一个外部类对象实例的引用，也就是 Activity.this，可以在 handleMessage() 的时候用于访问外部对象。

此时，如果 Handler 的 MessageQueue 里还有消息没有处理完，就会形成一条：

```
Message → Handler → Activity
```

的引用链。即使这时候 Activity 被销毁，也无法被回收，就发生了内存泄漏。

**解决方法是：**

1. **把 Handler 改成静态内部类**：静态内部类不会再依赖于外部的实例。但是这时候他也无法再访问外部实例了。
2. **因此，再使用弱引用 WeakReference 持有 Activity**：这样当 Ac 被回收的时候就可以被 GC 回收。

---

### Android 的 ART 的 GC 和 JVM 的 GC 区别是？

**首先是运行环境不同。**

JVM 主要运行在服务器环境，更关注吞吐量；而 ART 运行在移动设备上，更关注低卡顿和低功耗，所以会尽量减少 STW 时间。

**第二是 GC 结构不同。**

JVM 采用比较典型的分代 GC，新生代和老年代结构明显，常见收集器有 CMS、G1 等。而 ART 并不是严格的分代模型，主要有 Sticky GC、Partial GC、Full GC，通过缩小回收范围来减少 GC 停顿。

**第三是优化目标不同。**

JVM 的 GC 更关注**整体吞吐量**，虽然像 CMS、G1 也有并发阶段，但仍然可能出现相对较长的 STW。而 ART 的 GC 更强调并发回收，尽量把 STW 控制在很短时间，避免 Android 出现 UI 卡顿。

**总结：** JVM 的 GC 偏向服务器吞吐量优化，而 ART 的 GC 偏向移动设备的低暂停时间优化。

---

### Sticky GC、Partial GC、Full GC 的作用域？

ART 的 GC 主要基于 CMS（Concurrent Mark Sweep）思想实现，作用域 java heap。

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

### LeakCanary 的原理？

通过监听 Activity/Fragment 的 onDestroy，用弱引用监听已经调用 onDestroy() 但没有被 GC 回收的 Activity。

通过 `WeakReference` + `ReferenceQueue` 判断对象是否真的被 GC 回收，如果检测到没有回收，就会自动 dump heap 导出堆信息并提示可能的泄漏对象。

**① 监听生命周期：** Activity/Fragment 执行 onDestroy() 时，理论上应该被回收。

**② 弱引用监控：** LeakCanary 用 `WeakReference` + `ReferenceQueue` 观察对象是否被 GC 回收。如果这个 Activity 没有被其他对象引用，在下一次 GC 时就会被回收。

**③ 未回收则泄漏：** 一段时间后 WeakReference 没进入 ReferenceQueue → 对象仍被引用 → 自动 dump heap → 分析泄漏链。

**具体流程大致分几步：**

**第一步：监听 Activity / Fragment 的生命周期**

LeakCanary 会通过 `Application.registerActivityLifecycleCallbacks` 监听 Activity 生命周期。监听到 Activity 或 Fragment 执行 onDestroy() 时，说明这个对象理论上应该可以被回收了。

**第二步：使用 WeakReference 监控对象**

LeakCanary 会把这个 Activity 包装成 `WeakReference`，并放进一个监控集合。因为 `WeakReference` 不会阻止 GC 回收对象。如果这个 Activity 没有被其他对象引用，在下一次 GC 时就会被回收。

**第三步：通过 ReferenceQueue 判断是否被回收**

LeakCanary 会配合 `ReferenceQueue`：

- 如果对象被 GC 回收
- 对应的 `WeakReference` 会进入 `ReferenceQueue`

所以如果 onDestroy() 一段时间后这个引用没有进入队列，说明：**这个对象仍然被其他对象持有引用，可能发生了内存泄漏。**

---

### 内存回收中使用弱引用的底层原理？和虚引用的区别是？

在 JVM 可达性分析过程中，`WeakReference` 的 `referent` 不会被当作强可达路径参与对象图遍历；当对象只被弱引用关联时，会被判定为弱可达对象，并在 GC 的处理阶段被回收，同时清空 `WeakReference` 的 `referent`，如果绑定了 `ReferenceQueue`，则会将该引用入队。

| 引用类型 | 说明 |
|---------|------|
| **WeakReference** | 对象在 GC 判定为弱可达时，JVM 会先清空 `referent`，然后再入队。此时对象已经没有可访问引用，基本等价于"对象已经被清理/即将被回收"的通知 |
| **PhantomReference** | 对象在**即将被真正回收前**会被入队，此时 JVM 保证对象不会再被访问或复活，主要给程序一个机会去做资源清理（比如释放堆外内存） |

> 所以可以理解：
> - **WeakReference：** 对象被判定可回收后的通知
> - **PhantomReference：** 对象真正回收前的最后通知

---

### 具体怎么排查内存泄漏问题？

**一般分三步：**

1. **先通过内存监控发现问题：** 怀疑某个功能有内存问题，先通过 adb 查看进程内存使用情况。`adb shell dumpsys meminfo <包名>`。通过 meminfo 里的 PSS、Java Heap、Native Heap 来判断是 Java 层泄漏还是 Native 层问题。反复操作某个场景，比如反复进入退出某个 Activity 或页面。如果发现 Java Heap 持续增长、多次 GC 后内存仍然没有下降，说明可能存在对象没有被正确释放。
2. **使用 Heap Dump 或 LeakCanary 定位泄漏对象：** 确认可能存在泄漏后，可以导出堆信息。如果是开发过程中，可以接入 LeakCanary 辅助定位泄漏。
3. **通过引用链分析 GC Root 找到具体原因。**

---

### 弱引用处理事情有风险吗，如何避免？

有风险，因为 **弱引用随时可能被 GC 回收**。
比如：`Activity activity = weakRef.get();` 可能返回 null。

**避免方式：使用前判空**

```java
if (weakRef.get() != null) {
    weakRef.get().updateUI();
}
```

**短时间持有强引用**

```java
Activity activity = weakRef.get();
if (activity != null) {
    // 在当前方法作用域内是强引用
}
```

这样在方法执行期间不会被 GC。

---

### 强制判空导致业务停止怎么办

属于设计逻辑的问题。
弱引用只用于 UI 刷新，是因为 UI 生命周期不稳定，可能随时销毁；
而核心业务逻辑必须独立于 UI 存在，这样即使弱引用被回收，业务流程也不会受到影响。

---

### 弱引用用于 UI 更新一半被回收怎么处理？

**语义上的"保持局部强引用"。** 防止被回收。

当你这样写：`Activity activity = weakRef.get();`
发生了两件事：

1. weakRef.get() 取出对象引用
2. 这个引用被赋值给局部变量 `activity`

而 **局部变量默认是强引用（Strong Reference）**。

---

## 异步通信

### 主线程为什么不能做耗时操作，会出现什么问题，超过多久会出问题？

Android 的主线程（UI 线程）负责 UI 绘制、用户输入事件处理、组件生命周期回调。主线程内部通过 Looper + MessageQueue 按顺序处理消息，是单线程模型。

如果在主线程耗时操作，可能会：

- 阻塞 MessageQueue
- 导致新的 UI 绘制和输入事件无法及时处理
- 界面无法刷新

因此会出现**界面卡顿、点击无响应**等问题。严重时触发 **ANR（Application Not Responding）**。

**重要时间点：**

- **16ms：** 屏幕 60Hz 刷新率 → `1000ms / 60 ≈ 16.6ms`。如果 UI 绘制超过 16ms 未完成，就会掉帧、产生卡顿。
- **ANR 判定时间：**
  - Input 事件：5 秒
  - BroadcastReceiver：10 秒
  - Service：20 秒

---

### 如果需要执行一些耗时的操作，你会怎么做？

耗时操作需要放到子线程执行，避免阻塞主线程导致 ANR，任务完成后再切换到主线程更新 UI。

在实际开发中一般会使用线程池、Handler 或其他特性（例如协程）来管理异步任务，避免频繁创建线程带来的开销。

---

### Handler 的通信原理，如何判断 MessageQueue 为空？

Handler 本身是个进程内通信的作用，比如说主线程加载 UI，子线程执行耗时操作时候的通信。

**核心结构为：** Looper + MessageQueue + Handler

子线程通过 `sendMessage()` 将消息加入 MessageQueue，Looper 不断取出消息并分发到 Handler 的 `handleMessage()`，对应执行事件。

**判断队列是否为空：** 检查 `MessageQueue.peek()` 是否返回 null。

**扩展理解：**

- Looper 主线程维护，内部是一个循环，不断调用 `queue.next()`。
- MessageQueue 是单链表结构，按时间顺序存储消息。
- 空队列时 `next()` 会阻塞等待。

---

### Handler 中有 Loop 死循环，为什么没有阻塞主线程？原理是？

**死循环存在的必要性：**

ActivityThread 主线程本质上是一个一直在运行的程序。`Looper.loop()` 的死循环是为了保证主线程一直存活，持续接收并处理用户的交互和系统事件。如果没有这个死循环，主线程执行完 main() 之后就退出，app 就关闭了。

**Looper 的死循环没有阻塞主线程的原理是：**

这个死循环不会导致 CPU 资源被耗尽。因为 Android 的消息机制在底层依赖了 **Linux 的 epoll 机制**，实现了休眠与唤醒机制。有任务就执行，没任务就休眠。

当 MessageQueue 里面没有消息，调用底层方法进行休眠状态，让出 CPU 资源。当 MessageQueue 插入新消息，在调用方法唤醒主线程处理。

---

### 子线程能否更新 UI？

从功能上来说：通常情况下不能，即使能也不允许这么做。

Android 的 UI 空间是非线程安全的，多线程并发修改可能导致 UI 状态混乱。所以限制只能在主线程更新 UI，每次更新 View，底层会调用 `ViewRootImpl.checkThread()` 检查当前执行的线程是否是主线程，不是就会直接抛出异常。

因此，也会存在特殊情况：如果在 onResume 之前，ViewRootImpl 还没有实例化。理论上这个阶段是可以修改的，不会触发检查。但是实际开发中不允许这么做，因为 UI 更新的时间不可控，如果超过了 Resume 阶段可能导致崩溃。

另外，对于特殊 View 组件，比如 SurfaceView，它们是有独立的渲染机制，专门用于游戏或视频等高性能渲染，是可以在子线程中进行 UI 绘制的更新的。

---

### 为什么在子线程中创建 Handler 会抛出异常？

**一句话表示：**

Handler 的消息处理依赖于 Looper，子线程是没有自己的 Looper 的。主线程构造 new Handler() 的时候，内部会获取 ActivityThread 启动时自动创建的 Looper，而子线程是没有这个的，所以创建的时候会抛出异常。

如果要创建 Handler，需要先给子线程创建个 Looper。

---

### 跨进程通信的方式有哪些，Binder 相对于其他通信方式的优点在哪里？

Socket 是通用的进程间通信方式，甚至可以跨设备，但在 Android 里性能开销大；
Binder 是 Android 特有的 IPC，效率和安全性好，系统服务几乎都用它。

**扩展理解：为什么 Socket 开销大？**

Socket 开销大，是因为它走完整 TCP/IP 协议栈，涉及多次拷贝、上下文切换和协议解析； Binder 是 Android 特有的 IPC，通过内核驱动，路径短、拷贝少、调度少，因此性能远优于 Socket。

**用户态（User Mode）：** 应用程序运行的地方，权限低。
**内核态（Kernel Mode）：** 操作系统运行的地方，权限高（调度、内存、驱动）。
用户态 ↔ 内核态切换：需要 CPU 切换模式、保存寄存器、切换栈，是昂贵操作。

Socket 慢、Binder 快的核心原因之一就是：Socket 需要频繁用户态 ↔ 内核态切换，而 Binder 设计得更少、更轻

---

### Socket 通信机制和 Binder 对比？

**一句话对比：** Socket 走完整 TCP/IP 协议栈，每次 send/recv 都要用户态 ↔ 内核态切换，拷贝多、调度多，所以慢。Binder 是本地 IPC，通过 mmap 共享内存减少拷贝和切换，驱动直接调度线程池，所以快。

---

### 先搞清楚：用户态 / 内核态是什么？是 CPU 的运行模式

- **用户态：** App、Java 代码运行的地方，权限低
- **内核态：** Linux 内核运行的地方，权限高（调度、内存、驱动）
- 用户态不能直接访问硬件，必须通过**系统调用（syscall）** 进入内核态。

---

### 什么是用户态 ↔ 内核态切换？

当你调用：

- `send()`
- `recv()`
- `read()`
- `write()`
- `ioctl()`

这些都是**系统调用**，会触发 CPU 主动从用户态切换到内核态。

1. 保存寄存器、切换栈、切换页表
2. 执行内核代码
3. 再切回用户态

这是**昂贵操作**。只有"用户态主动 syscall → 内核态"才算一次切换成本。内核调度导致的用户态 ↔ 内核态切换不算。

用一个请求-响应的流程来对比：

---
### Socket vs Binder：为什么切换多？

一、Socket 通信流程

Socket 属于网络通信模型，即使是本机通信（localhost），也需要经过 TCP/IP 协议栈。

一次完整的 **请求 → 响应** 通信涉及：

- **4 次 syscall**（request 2 次 + reply 2 次）
- **8 次上下文切换**
- **4 次数据复制**

① 发送请求：Client 发起

```
应用调用：socket.getOutputStream().write(bytes)
          ↓
Java 层通过 JNI 调用 libc 的 send()
          ↓
系统调用，CPU 从用户态 → 内核态
```

内核随后执行 TCP/IP 协议栈处理，数据拷贝：

```
client user memory
        ↓
   copy_from_user
        ↓
  client kernel socket buffer
```

Syscall 返回：内核态 → 用户态

② 内核协议栈处理

本机通信时，数据不经过网卡，在内核协议栈中直接转发：

```
client kernel socket buffer
        ↓
   TCP/IP 协议栈处理
        ↓
server kernel socket buffer
```

**完全发生在内核态内部，不涉及用户态切换。**

③ 服务端接收数据

服务端调用 `socket.getInputStream().read()`，底层调用 `recv()`，再次触发 syscall：

```
CPU 从用户态 → 内核态

server kernel socket buffer
        ↓
      copy_to_user
        ↓
  server user memory
```

Syscall 返回：内核态 → 用户态

④ 服务端处理完成 → 返回响应

流程与 Request 完全对称。

完整数据路径

```
client user memory
        ↓
   copy_from_user
        ↓
  client kernel socket buffer
        ↓
   TCP/IP 协议栈
        ↓
  server kernel socket buffer
        ↓
      copy_to_user
        ↓
  server user memory
```

---

二、Binder 通信流程

Binder 的开销：

- **2 次 syscall**（而非 4 次）
- **4 次上下文切换**（而非 8 次）
- **2 次数据复制**（而非 4 次）

① 发起事务：Client 用户态 → 内核态

```
Client 调用：client → Proxy.transact()
          ↓
  Java → JNI → libbinder → Binder driver
          ↓
  将参数序列化到 Parcel，位于 Client 用户态内存
```

执行 `ioctl(BINDER_WRITE_READ)`：

- **第一次 syscall**：CPU 从用户态 → 内核态
- Binder 驱动通过 `copy_from_user` 把数据从 Client 用户态内存拷贝到 Binder driver buffer（内核空间）

② Binder 驱动调度服务端线程
服务端进程通常有一个 Binder 线程池，这些线程平时阻塞在内核态等待新的 Binder 事务。 

当 Binder driver 收到客户端事务后：

- 将事务加入服务端message队列 （）
- 
- 唤醒一个 Binder 线程

Binder 线程从 ioctl 返回，CPU 从 内核态 → 用户态。 

由于该 Binder driver buffer 在服务端初始化时已经通过 mmap 映射到服务端用户空间，

因此 Binder 线程可以在用户态直接访问这块内存中的数据 

Stub 再从 Parcel 中反序列化参数并调用真正的服务实现 Stub.onTransact()。

如果是核心业务（如 AMS/WMS），Binder 线程会将请求包装成一个 Message 对象，通过 Handler

投递到主线程的 MessageQueue 后回到内核态继续等待即可。







③ 服务端执行完毕 → 发送 Reply

流程对称，执行**第二次 syscall**，返回结果给 Client。

---

三、核心差异

Binder 的优化点

> **不是消灭 copy，而是消灭"多余的 kernel buffer 和协议栈"。**

```
Socket:  user → kernel → kernel → user
Binder:  user → kernel → user
```

| 维度 | Socket | Binder |
|------|--------|--------|
| syscall 次数 | 4 | 2 |
| 上下文切换 | 8 | 4 |
| 数据复制 | 4 | 2 |
| 经过内核 buffer | 2 个（client + server） | 1 个（Binder driver） |
| 协议栈 | TCP/IP | 无 |

---

### AIDL 是什么？怎么使用？

AIDL 只是定义接口的规范，用来生成 Binder 通信所需的组件。

**流程：**

1. **定义接口文件：** 在 `.aidl` 文件里写接口方法签名。
2. **编译生成代码：** 系统自动生成接口、Stub（服务端实现用）、Proxy（客户端代理）。
3. **服务端实现接口：** 在 Service 里继承 Stub，实现具体逻辑，并在 `onBind()` 返回这个对象。
4. **客户端绑定服务：** 用 `bindService()` 获取 IBinder，再通过 `Stub.asInterface()` 转换成接口对象。
   - 同进程：直接返回 Stub。
   - 跨进程：返回 Proxy。
5. **Binder 驱动传输：** Proxy 调用方法 → Binder 驱动传输数据 → 服务端 Stub 执行逻辑。

**1. 定义接口文件**

在 `*.aidl` 文件里写好接口方法签名，比如：

```java
interface IMyAidlInterface {
    void sendMessage(String msg);
}
```

**2. 编译生成代码**

Android 编译器会自动生成对应的 Java 文件，包含 Stub/Proxy 类。

- **Stub：** 服务端实现用的抽象类。
- **Proxy：** 客户端调用用的代理类。

**3. 服务端实现接口**

在 Service 里继承 Stub 创建一个对象，重写实现具体方法逻辑。然后在 `onBind()` 返回这个对象。

这里的 `IMyAidlInterface.Stub` 是编译 AIDL 文件后自动生成的抽象类，它继承了 Binder 并实现了接口。

```java
public class MyService extends Service {
    private final IMyAidlInterface.Stub mBinder = new IMyAidlInterface.Stub() {
        @Override
        public void sendMessage(String msg) {
            // 处理消息
        }
    };

    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
}
```

**4. 客户端绑定服务**

用 `bindService()` 连接到服务端，拿到 `mBinder` 对象。客户端调用 `Stub.asInterface(service)` 时，系统会判断：

- 如果在同一进程，直接返回 Stub。
- 如果是跨进程，返回 Proxy 对象。

**5. Binder 驱动传输数据**

如果是通过 Proxy 调用远程方法，底层其实是走 Binder 驱动。AIDL 定义的接口最终会通过 Binder 驱动在内核层完成跨进程通信。

---

### AIDL 通信是同步的还是异步的？为什么这么设计？

AIDL 默认是同步调用，因为 Binder 采用的是 RPC（Remote Procedure Call）设计思想，希望远程服务调用方式和本地方法一样简单。
客户端调用 AIDL 方法时是同步的，会阻塞等待服务端返回结果。很多系统服务也需要立即返回结果，比如 PackageManager、ActivityManager 等。

不过需要注意的是：客户端调用是同步的，但服务端并不是在主线程执行，而是在 Binder 的线程池（Binder Thread Pool）中处理请求。
也就是说，当客户端发起调用后，Binder 驱动会把请求分发给服务端进程中的某个 Binder 线程来执行，执行完成后再把结果返回给客户端线程。

另外 Binder 底层的 transaction 机制本身也是同步等待 reply 的模型。
如果需要异步调用，可以在 AIDL 方法前使用 oneway 关键字，这样客户端发送请求后会立即返回，不会等待服务端执行完成。

---

### Binder 服务连接不上有哪些原因？

首先是 Service 是否在 Manifest 中正确注册和声明权限。如果 Service 没有在 AndroidManifest.xml 中声明，系统是无法找到这个服务的，自然也就无法建立连接。如果声明了 permission，客户端应用必须拥有相同的权限，否则绑定会失败。

其次是 Intent 匹配和 exported 配置问题。如果通过 bindService 使用隐式 Intent 绑定，需要保证客户端的 action 和 Service 中 intent-filter 声明的 action 一致，否则系统无法匹配到目标 Service。
另外在 Android 12 之后，如果 Service 声明了 intent-filter 必须显式配置 `android:exported`，跨应用绑定时需要设置为 true，否则系统会拒绝连接。

---

### Handler 的 postDelay 为什么切到后台就不生效了？

Handler 的 `postDelayed` 本质是把 Runnable 封装成 Message，并计算执行时间 `when = SystemClock.uptimeMillis() + delay`，然后按时间顺序插入 MessageQueue。

Looper 不断调用 `MessageQueue.next()`，当 `当前 uptimeMillis >= message.when` 时就取出执行。

由于 `postDelayed` 使用的是 `SystemClock.uptimeMillis()` 作为时间基准，而 `uptimeMillis` 在设备 deep sleep 时不会增长。当应用进入后台、设备进入 sleep 时计时会暂停，因此任务会被明显延迟，看起来像 `postDelayed` 失效。

> 因此：Handler 只适合进程存活期间的短期调度。如果需要在后台或设备休眠期间仍然按时执行，通常需要使用 AlarmManager 或 WorkManager。

---

### AlarmManager、WorkManager、JobScheduler 的区别和联系？

**AlarmManager** 是早期 Android 中常用的定时任务机制，可以在指定时间或周期触发一个 PendingIntent（例如启动组件或发送广播）。优点是定时比较精确、使用简单。但缺点是容易被应用频繁使用唤醒设备，比较耗电，而且在 Doze 等省电机制下很多任务会被系统延迟执行。

后来 Android 引入了 **JobScheduler**，可以为任务设置执行条件，比如需要网络、设备充电或者设备空闲等。启动一个 `JobService`，条件满足后回调的方式执行任务，从而减少设备唤醒次数，更加省电。但 JobScheduler 只支持 Android 5.0 及以上版本。

为了解决不同 Android 版本兼容的问题，Google 在 Jetpack 中封装了 **WorkManager** 框架，它会根据系统版本自动选择底层实现：在高版本系统上使用 JobScheduler，在低版本上使用 AlarmManager。同时它还支持任务链、任务重试以及任务持久化等功能。

**总结来说：**

- **AlarmManager：** 主要用于定时任务，时间较精确
- **JobScheduler：** 系统级后台任务调度，可以设置执行条件
- **WorkManager：** Jetpack 提供的统一后台任务方案，内部适配 JobScheduler 和 AlarmManager，现在官方更推荐使用 WorkManager 来执行后台任务。

---

## 稳定性与性能

### 三大稳定性问题是什么？有什么区别？

**Java Crash：**

- 由 Java 层未捕获异常导致。
- 常见原因：NullPointerException、IndexOutOfBounds、IllegalState。
- 日志来源：logcat。

**Native Crash / Tombstone：**

- 由 C/C++ 层错误导致，例如野指针、数组越界、内存非法访问。
- 系统会生成 tombstone 文件，包含寄存器、堆栈、内存快照。
- 日志来源：`/data/tombstones/`。

**ANR：**

- 应用主线程长时间阻塞，无法响应用户操作。
- 常见原因：主线程执行耗时操作、死锁。
- 日志来源：`/data/anr/traces.txt`。

---

### 如果线上出了 ANR 问题，怎么排查？

排查线上 ANR 问题，需要分系统层和应用层两个视角来看：

**如果是系统层**（比如手机厂商视角来看），就比较直接：

直接抓一下系统底层的 Dropbox 日志。ANR 发生时，AMS 会把对应的 trace、log、cpu 负载都记录下来，直接通过 dropbox 日志来分析即可。

**如果是应用层**（三方 APP 类），因为没有系统权限，无法读取系统日志：

通常需要接入一些 APM 的监控（比如 matrix、bugly）的包。通过开启子线程的 watchdog 机制，或者在底层拦截 SIGQUIT 信号，自己抓取主线程堆栈并上报。

---

### ANR 问题的本质是什么？

ANR 的本质就是**"超时检测机制"**：系统在关键组件（比如 ActivityManagerService、InputDispatcher、BroadcastQueue 等）里写死了固定的响应时间常量，如果超过这个时间没有响应，就会触发 ANR。

如：

- 输入事件：5 秒内主线程必须处理，否则报 ANR。
- Service：前台服务 20 秒，后台服务 200 秒。
- BroadcastReceiver：前台广播 10 秒，后台广播 60 秒。

受设备负载影响：在高 CPU 占用、内存紧张、IO 卡顿时，主线程可能无法在规定时间内完成任务 → 超时 → ANR。

**真正原因要看日志具体确认。**

---

### 如何排查和解决 UI 卡顿问题？

Android 的 UI 渲染依赖于一条基于 VSYNC 信号的流水线：

1. **信号源头：** 底层屏幕硬件按固定频率发出 VSYNC 信号，比如 60Hz 约为 16.6ms
2. **信号分发：** SurfaceFlinger 接收信号并分发给前台 APP
3. **接收绘制：** APP 主线程的 Choreographer 接收到信号后，触发 `doFrame` 开始绘制这一帧，由 RenderThread 驱动合成

**卡顿的根本原因：**

从应用侧看，如果应用主线程在 16.6ms 没有完成一帧的渲染，用户可能就会感受到卡顿。

**应用层（App 侧）：数据准备与指令生成超时**

这是日常开发中最常见的卡顿原因。

- **主线程阻塞：** 主线程执行 doFrame 耗时过长，例如 UI 嵌套过深导致 Measure/Layout 缓慢，或者在主线程进行了 IO、锁等待、引发频繁 GC。
- **RenderThread 阻塞：** 主线程虽按时交出指令，但 RenderThread 在同步数据或上传超大 Bitmap 纹理时耗时过长。

**系统合成层（SurfaceFlinger 侧）：图层合成失败**

App 侧按时输出了画面数据，但在系统级的图层管理和合成阶段出现瓶颈。

- **HWC 退化：** 屏幕上的图层数量过多或有特殊混合模式，导致硬件合成器（HWC）处理失败，退化为耗时的 GPU 合成（Client Composition）。
- **SF 调度异常：** 系统整体负载极高，导致 SurfaceFlinger 自身线程被抢占，错过了 VSYNC-SF 信号，无法按时将画面送显。

**硬件渲染层（GPU 侧）：图形计算超载**

指令和合成调度都正常，但底层 GPU 算力跟不上。

- **过度绘制（Overdraw）：** 同一像素点在单帧内被反复绘制，浪费算力。
- **复杂特效：** 大量使用复杂 Shader、圆角裁剪或模糊效果，超出 GPU 处理能力。
- **硬件降频：** 设备发热触发温控，导致 GPU 降频，渲染能力大打折扣。

---

### SharedPreferences 为什么容易导致 ANR？有什么替代方案？

SharedPreferences 的 `commit()` 会在主线程同步写磁盘，容易导致 ANR。`apply()` 虽然是异步写入，但仍然存在锁竞争和磁盘压力问题，在高频或大数据场景下也可能卡顿。

**官方替代方案：DataStore**

- 优点：
  - 基于 Kotlin 协程和 Flow，线程安全。
  - 提供两种模式：Preferences DataStore（键值对存储）、Proto DataStore（强类型存储）。
  - 异步、非阻塞，避免主线程 I/O。

**工程实践替代方案：MMKV**

- 优点：
  - 腾讯开源的高性能键值对存储库。
  - 基于 mmap 内存映射，读写性能远高于 SharedPreferences。
  - API 接口与 SharedPreferences 类似，迁移成本低。

---

## 设计、架构与 JetPack

### Android 层级结构（带 HAL）

1. **应用层（Application Layer）**
   - 我们写的 App：Activity、Fragment、Service。
2. **应用框架层（Application Framework）**
   - 提供各种系统服务：ActivityManager、ContentProvider、NotificationManager。
3. **系统运行库层（Libraries & Android Runtime）**
   - 包含 C/C++ 库（比如 SQLite），以及 ART 虚拟机。
4. **HAL 层（Hardware Abstraction Layer）**
   - 介于 Framework 和内核之间。
   - 提供统一接口，比如 Camera HAL、Audio HAL、Sensor HAL。
   - 上层只管调用 HAL API，不需要关心具体硬件驱动实现。
5. **Linux 内核层（Linux Kernel）**
   - 提供进程管理、内存管理、硬件驱动。
   - HAL 最终会调用内核驱动来操作硬件。

---

### MVC、MVP 和 MVVM 有什么本质区别？为什么现在 Google 官方推荐使用 MVVM？

MVC、MVP 和 MVVM 的本质区别在于三者如何解耦 View 与 Model：

- **MVC：** 中 Activity/Fragment 同时承担 View 和 Controller，耦合度高；常见的写法是所有代码都写到 Activity 里面，也不利于管理。
- **MVP：** 引入 Presenter，切断 View 与 Model 的直接联系，但通过 Presenter 来打通 Model 和 View 之间的数据传递，主要依赖接口的实现；如果接口中的某个功能设计修改，还需要把实现它的各部分都修改一次，小修改变成了大工作量。
- **MVVM：** 引入 ViewModel，结合数据绑定（LiveData/Flow/Compose），ViewModel 持有 LiveData，View 监听 LiveData 的变化，实现数据流驱动、响应式更新。并且 ViewModel 的生命周期独立于 Activity，更安全。

Google 推荐 MVVM，是因为它更符合现代 Android 的响应式编程理念，更好的解耦功能，提升可维护性和测试性。

---

### MVVM 架构中，Model、View、ViewModel 各自的职责是？他们之间的依赖关系是怎样的？谁持有谁的引用？

**Model：**

数据层。负责 API 调用、数据库读写、业务规则。不感知 UI 存在。

**ViewModel：**

状态管理层。持有 View 需要的数据，以可观察数据流（LiveData/StateFlow）形式暴露给 View。负责协调 Model 和 UI 状态的映射。不持有 Android 框架引用。

**View：**

渲染层。负责页面展示和用户交互，把事件转发给 ViewModel。不包含业务逻辑。

**依赖关系：**

View → ViewModel → Model，单向依赖，ViewModel 不反向持有 View。

**谁持有谁：**

- **ViewModel 持有 Model**——调数据
- **View 持有 ViewModel**——观察数据、触发事件
- **ViewModel 不持有 View**——只暴露数据流，不直接指挥 UI

这是和 MVP 的根本区别：

- **MVP 的 Presenter** 持 View 接口直接调用（`new Presenter(this.Activity)`）
- **MVVM 的 ViewModel** 只暴露数据让 View 自己观察，不需要接口桥梁。

---

### ViewModel 为什么能在屏幕旋转导致 Activity 重建时存活下来？内部原理是什么？

- **普通成员变量：** 随 Activity 一起销毁，旋转后数据丢失。
- **ViewModel：** 存放在 ViewModelStore，通过 `onRetainNonConfigurationInstance()` 保留，旋转后复用。

**关键类：** ComponentActivity（现在的 Activity 会继承它）。

- ComponentActivity 内部持有一个 ViewModelStore。ViewModelStore 挂在 NonConfigurationInstance 容器上，不会因为 Activity 销毁而消失。
- 在 `onRetainNonConfigurationInstance()` 方法里，系统会保存这个 Store。
- 当新的 Activity 创建时，会通过 `getLastNonConfigurationInstance()` 拿回旧的 Store。
- 所以 ViewModel 并没有被销毁，而是被复用。

**本质：** ViewModel 生命周期绑定的是**系统的生命周期容器**，而不是 Activity 本身，独立于 Activity 的生命周期。

**一句话收尾：**

> ViewModel 能在屏幕旋转时存活，是因为它存放在 ViewModelStore 中，系统在配置变化时保留并复用这个 Store，从而保证数据不丢失。

---

### 如果在 ViewModel 中需要使用 Context，应该怎么做？为什么绝对不能在 VM 中传入 Activity 的 Context？

**首先，不能传入是因为：** 生命周期的差异可能导致内存泄漏。

**生命周期不一致：**

- Activity 的生命周期很短，可能因为旋转屏幕或被系统回收而销毁。
- ViewModel 生命周期比 Activity 长（通常和宿主的生命周期绑定），如果持有 Activity Context，就可能在 Activity 已销毁时仍然引用它。

**内存泄漏风险：**

- ViewModel 持有 Activity Context → Activity 无法被 GC 回收 → 内存泄漏。
- 特别是在配置变化（屏幕旋转）时，新 Activity 创建，旧 Activity 销毁，但 ViewModel 还在引用旧的 Context。

**如果确实需要 Context（比如访问资源、系统服务），应该使用 Application Context。**

- 官方提供了 AndroidViewModel，它继承自 ViewModel，并在构造函数里传入 Application，这样你就能安全地拿到 `getApplication()`。
- Application Context 生命周期和应用进程一致，不会因为 Activity 销毁而泄漏。

**避免直接传 Context：** 如果只是为了拿字符串、颜色等资源，推荐通过 ResourceProvider 或封装工具类，而不是直接把 Context 塞进 ViewModel。

---

### LiveData 的核心作用是什么？它是如何做到生命周期感知的（Lifecycle-Aware）？

**核心作用：** 作为一个可观察的数据容器，数据变化之后自动通知 View 更新。实现数据流驱动的功能。

**生命周期感知能力：** 是指他能避免因 Activity/Fragment 停止导致的内存泄漏等问题。

LiveData 通过 `LifecycleBoundObserver` 监听 LifecycleOwner 的生命周期事件，而 Activity 和 Fragment 都实现了 LifecycleOwner 接口，所以他能够只在组件活跃状态分发数据，销毁时自动解绑，从而实现 Lifecycle-Aware。

---

### ViewModel 不能持有 View 的引用。那么当 VM 需要让 View 弹出一个 Toast 或者页面跳转时，怎样实现一次性消费？

**为什么不能直接在 VM 里调用 Toast/跳转：**

- **职责分离：** ViewModel 只负责数据和业务逻辑，不负责 UI 操作。
- **生命周期问题：** 如果直接持有 Activity/Fragment 引用，可能导致内存泄漏。

**一次性消费的常见实现方式：**

**1. SingleLiveEvent：**

本质是对 LiveData 的扩展，只分发一次事件。避免了配置变化（比如旋转屏幕）时重复消费的问题。

```java
public class SingleLiveEvent<T> extends MutableLiveData<T> {
    private final AtomicBoolean pending = new AtomicBoolean(false);

    @Override
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
        super.observe(owner, t -> {
            if (pending.compareAndSet(true, false)) {
                observer.onChanged(t);
            }
        });
    }

    @Override
    public void setValue(T t) {
        pending.set(true);
        super.setValue(t);
    }
}
```

**2. Event 封装 + LiveData：**

定义一个 `Event<T>` 包装类，标记数据是否已消费。ViewModel 发出 Event，UI 层判断是否已消费，避免重复。

```java
public class Event<T> {
    private final T content;
    private boolean hasBeenHandled = false;

    public Event(T content) { this.content = content; }

    public T getContentIfNotHandled() {
        if (hasBeenHandled) return null;
        hasBeenHandled = true;
        return content;
    }
}
```

**3. SharedFlow (Kotlin)：**

在 Kotlin + Flow 场景下，推荐用 SharedFlow 来实现一次性事件。比 LiveData 更灵活，支持背压和协程。

---

### 相比于传统的 LiveData，现在很多使用 Kotlin 的 StateFlow/SharedFlow，他们有什么区别吗？

**LiveData：**

Jetpack 组件，自动感知 LifecycleOwner 接口，只在活跃状态分发数据，避免内存泄漏。适合传统 XML + ViewModel 项目。

**StateFlow：**

Kotlin 协程原生的状态流，必须有初始值，始终持有最新状态。更适合表示 UI 状态（如登录态、加载状态）。

**SharedFlow：**

不持有单一状态，而是事件流。可配置 replay（事件缓存数量），支持多个观察者同时订阅。适合一次性事件（Toast、导航、弹窗），或者多观察者需要同时消费的场景。

---

### MVVM 的体现场景？

以平时接触的笔记应用中，通过账号登录的逻辑为例：

**整个流程是：**

- 用户点击上层的 View 登录
- ViewModel 收到指令去调 Model（账号的 SDK）
- Model 拿到账号信息返回给 ViewModel
- ViewModel 更新内部的 LiveData
- 由于 View 一直在监听 LiveData 的数据变化，刷新 UI

**这种做法最大的好处是解耦**，UI 界面和登录的 SDK 分离，中间通过 ViewModel 来更新。

---

### JVM、Dalvik、ART 三者的原理和区别？

主要是**运行文件、底层架构、编译时机**的三方面区别：

**1. 运行的文件不同：**

JVM 是 PC 端，运行标准的 `.class` 文件。Dalvik 和 ART 都是专门为移动端压缩过的 `.dex` 文件。去掉了冗余信息，为了给手机省内存。

**2. 底层的架构不一样：**

JVM 是基于栈计算的，Dalvik 和 ART 是基于寄存器的，更适合手机 CPU，效率高。

**3. 最核心的区别：编译代码的时机不同。**

Dalvik 是早期使用，当时 Android 手机的硬件存储空间不大，只能以时间换空间。APP 每次运行的时候，需要实时把 dex 转换为机器码，就会有启动慢、耗电的问题。

ART 是现在的标配，内存空间已经不再是瓶颈问题，可以用一定空间换时间。APP 安装的时候就把代码编译成机器码，安装稍慢、但运行打开更快。

---

### MMKV、DataStore、Room 是什么？有什么区别？

总的来说，MMKV 和 DataStore 都是 K-V 存储的数据库，Room 是 SQLite 的一种封装方案。

**MMKV：** 是腾讯开源的 K-V 存储库，是 SharedPreferences 的一种高效替代方案。优势在于他的读写时直接在内存操作，然后同步到文件，所以性能更好，也支持多线程访问。适合存一些用户状态、开关配置这种简单数据。

**DataStore：** 是 Google Jetpack 里面推出的 SharedPreferences 替代方案。特点是基于 Kotlin 协程和 Flow，保证读写都是异步并且安全的。

**Room：** 和前两个不太一样，他是 SQLite 的一层封装，简化了 SQL 的使用，增加了类型安全检查、异步支持等功能。适合存储结构化数据。

---

### JetPack 的 Room 数据库相比于原生 SQLite 有什么优势？

**SQLite 的缺点：**

1. **API 繁琐：** 需要手写 SQL，管理 Cursor，容易出错。代码冗长，维护成本高。
2. **类型安全不足：** SQL 语句是字符串，编译期无法检查错误。错误只能在运行时暴露。
3. **生命周期管理复杂：** 数据库操作容易和 UI 生命周期耦合，导致内存泄漏或崩溃。需要手动管理线程。
4. **异步支持不足：** 原生 API 没有协程或响应式支持。需要自己封装线程池或 AsyncTask。
5. **数据库升级麻烦：** 需要手动写升级 SQL，容易出错。

**Room 在这些缺点上的优化：**

> Room 是 SQLite 的升级版，底层仍然用 SQLite，但在 API 简化、类型安全、生命周期管理、异步支持、升级机制 等方面做了全面优化。

---

### 为什么 MMKV 比 SharedPreferences 快？

MMKV 对 SharedPreferences 的性能碾压，主要体现在底层读写机制和数据写入策略两个核心维度的差异：

**1. 底层机制：mmap 内存映射 vs 传统 I/O**

- **SharedPreferences：** 使用传统的 I/O 操作。每次写盘都需要经历数据从用户空间拷贝到内核空间，再写入物理磁盘的过程，系统开销较大。
- **MMKV：** 核心黑科技是使用了 mmap（内存映射）技术。它将磁盘上的物理文件直接映射到 APP 的内存地址空间。APP 对这块内存的任何修改，就等同于直接修改了文件，彻底省去了数据在用户态和内核态之间的拷贝过程。
- **写盘时机：** MMKV 写入内存后，真正落盘的动作交由操作系统内核在后台自动调度（如 pdflush 线程）。这种机制极其高效，且即使 APP 发生 Crash，只要手机没断电，系统内核依然会保证将内存中的"脏页"安全刷入磁盘，不会丢失数据。

**2. 写入策略：增量追加 vs 全量覆盖**

- **SharedPreferences：** 基于 XML 格式。哪怕你仅仅修改了一个简单的布尔值，SP 也会在内存中重新构建整个 XML 结构，然后对磁盘文件进行全量覆盖写入。文件体积越大，修改耗时越长。
- **MMKV：** 采用类似 Protobuf 的紧凑二进制编码。当有新数据或修改数据时，MMKV 不需要重写整个文件，而是直接在文件末尾进行增量追加（Append）。只有当分配的内存/文件空间快满时，才会触发一次重写来剔除冗余的旧数据。这种追加模式极大地降低了 I/O 耗时。

SP 写盘不仅需要切态（用户态切内核态），还需要额外的数据拷贝；而 MMKV 利用 mmap 实现了直接在用户态写内存，省去了昂贵的上下文切换和拷贝开销，这也是它性能极高的核心底层原因之一！

---

## Kotlin

### Kotlin 线程和协程的区别？

简单来说，**线程是操作系统的底层资源，协程只是跑在线程上的轻量级任务**。

1. **本质与资源消耗：**
   - 线程是操作系统的底层资源，创建和销毁的开销大，占用内存多。
   - 协程是用户态控制，单机可以轻松创建大量的协程也不会 OOM，占用内存小。

2. **切换开销：**
   - 线程切换需要操作系统调度，上下文切换的维护成本高。
   - 协程切换只是由 Kotlin 代码级别的状态机流转就能控制，开销极低。

3. **等待机制（最核心区别）：**
   - 线程遇到耗时任务会被阻塞，导致线程资源被占用卡死。
   - 协程可以通过 `suspend` 关键字挂起任务，主动让出当前线程给其他协程使用，等结果返回后再恢复，提高了线程的利用率。

---

### Kotlin 协程和 Java 的虚拟线程对比？

协程和虚拟线程都是挂载在真实线程上执行任务的。本质上都是把海量的轻量级任务，交给少量的真实操作线程来执行实现并发。并且在遇到耗时操作的时候，要把真实操作线程让出来给别人用。

**在让出线程的底层机制上：**

1. **Kotlin 协程**靠编译器把代码编译成一个状态机，挂起时保存当前状态，恢复时再找个线程继续执行任务。
2. **Java 的虚拟线程**靠 JVM，阻塞时 JVM 把它的调用栈从线程复制到堆内存里暂存，阻塞结束后再把栈数据拷贝回真实线程继续执行。

---

### 协程具体是怎么实现的，Kotlin 实现协程的具体步骤？

Kotlin 协程的实现本质是编译器把 `suspend` 函数转换成状态机。

每个 `suspend` 标记的函数在编译后都会多一个 `Continuation` 参数，用来保存协程的执行状态。

当执行到挂起点时会保存当前状态并返回，线程被释放；等任务完成后通过 `Continuation.resume()` 恢复执行，根据状态机继续运行。

利用状态机和参数，实现用户态（代码级别）的协程管理。

---

### Suspend 关键字的作用是？

Suspend 关键字只是一个标记，用来说明一个权限。仅仅是告诉编译器，这个函数可以挂起当前协程。但是到底挂不挂起，取决于函数内部到底有没有执行真正的底层挂起逻辑。

真正的挂起点，是代码执行到真正需要等待时，才发生的动作。

---

### Kotlin 的 StateFlow 和 SharedFlow 的区别？

**StateFlow：** 顾名思义，用于维护状态。因此必须带一个状态初始值。由于状态可能有跳变，因此它自带防抖措施，连续发相同值会被过滤。新订阅者永远能拿到最新的状态值，比较适合 UI 刷新。

**SharedFlow：** 适合多个订阅者共享事件流。因此也可以没有事件，不需要初始值。类似于队列的形式，也不需要防抖，发多少收多少即可。新订阅者不会再收到旧的、被消费的事件，适合于一次性事件。

---

## 性能优化

### 如何对 Android 性能优化？

我主要从内存和稳定性两方面：

**1. 内存维度的看护（常驻内存与动态内存）**

与基线数值和竞品机对比是否超标，分析现象原因。

- **常驻内存优化：** 通过挂测关注核心进程的常驻内存。如果常驻内存过高，可能会导致系统频繁触发 LMK，杀掉用户的后台应用进程导致功能失效或者卡顿。一般会排查是否有不必要的常驻服务，是否加载了过大的无用资源，推动问题澄清或优化。
- **动态内存监控：** 通过用例挂测模拟用户使用场景，观察动态内存的波动。主要排查两点：一是内存泄漏，导致内存只增不减。二是判断内存抖动是否属于正常业务。

**2. 稳定性维度**

主要是系统的稳定性指标、进程稳定性 APR 问题：根据监控的数据、分析和处理稳定性问题。

比如对于 ANR 分析 dropbox 日志，看主线程到底是被 IO 阻塞、锁等待，还是被其他高优先级线程抢占了 CPU。

对于 Crash 和 Tombstone，通过打印堆栈分析代码中是否存在一些逻辑漏洞，再边缘场景可能会触发崩溃，是否影响用户使用场景。

---

### OOM 的场景可能是？

**首先从系统角度来看：**

要看看是不是 ART 本身为应用分配的虚拟机内存达到了上限，涉及到 ART 的参数。

**从应用角度来看：**

1. **Java 堆内存溢出：** 通常由内存泄漏积累导致 GC 无法回收，或者是瞬间有大对象分配，比如超大分辨率的 Bitmap，击穿了可用内存。
2. **Native 内存溢出：** 对于大量使用 JNI 的应用，比如音视频和游戏，native 层的内存问题比如分配了内存而没有 free 导致泄漏。此时整机内存可能会异常偏高。
3. **线程数超限制：** 代码逻辑中可能有死循环创建，每个线程都要在底层分配独立的栈内存导致溢出。
4. **文件描述符耗尽：** Linux 系统中每个进程能打开的文件描述符是有限的。可能是频繁打开资源但没有及时释放。

---

### ANR 的场景可能是？

本质总结为两个维度的阻塞：

1. **进程内阻塞：** 主线程的 MessageQueue 发生拥堵，导致系统分发的时间没有及时被消费。比如广播没有及时处理的后台 ANR，输入事件分发没有及时处理的前台 ANR。
2. **跨进程阻塞：** 主线程发起同步 Binder 调用，被对端阻塞，通常是 SystemServer 就阻塞了。

---

### ANR 的根本原因是？

**从系统角度来看，一般是整机资源不足：**

1. **系统资源匮乏：** CPU/GC/高温高负载
   - CPU 高负载，后台的大核被其他进程占用。
   - 频繁的 GC，内存严重不足，GC 过程 STW 可能导致主线程停顿，触发 ANR。
   - 高温，整机高温导致 CPU 降频，并且通常高温也伴随着高负载，导致了 ANR。

2. **从应用角度来看，一般是代码逻辑可能存在问题：**
   - 主线程中直接进行了大量 IO、数据库读写等复杂操作。
   - 主线程可能在等待子线程的操作、或者锁和网络请求等被阻塞，触发了 ANR。

---

### Android 中内存优化的方式有哪些？

**从系统进程的管理角度来看：**

1. **主动释放内存，降低被杀后再被拉起的次数：** 应用进程退到后台时，清理内部的数据缓存，用途是减低内存占用，避免被 LMK 查杀，换取下一次的热启动秒开，虽然加载资源可能会有一定资源使用，但相比冷启动已经轻量化许多了。
2. **治理关联拉起，切断不必要的进程创建：** 需要监控前台 APP 使用期间，后台是否有不必要的关联启动，过多的后台会瓜分系统内存，影响前台 APP 的使用。具体业务具体分析，分析是否有不必要的业务拉起，是否可以优化业务逻辑。

**从应用角度来看：**

1. 防止内存泄漏，针对可能发生内存泄漏的场景重点关注。
2. 高频使用的临时对象加缓存，避免频繁触发 GC 导致的卡顿。
3. 特殊对象比如 Bitmap 这种，重点关注内存占用。

---

### Android Native Crash 问题如何分析定位？

有大数据平台，一般去捞一下对应时间段的流水日志。

结合 dropbox、tombstone 这些系统日志，看下报错的堆栈和底层的 Signal 信号，判断问题原因。

---

### 代码混淆的作用和步骤是？

**最核心的作用有两个：**

1. 通过把代码变成 `a.b.c` 这种样式，反编译，提升代码安全。
2. 缩短了代码量，大幅减少 APK 的包体积。

**从开发步骤的角度看：**

1. **在 build.gradle 中开启反编译开关。** 混淆本身会拉低编译速度，在项目中通常只有在正式包才开启混淆，开发调试不会打开。
2. **在 proguard-rules.pro 中写好规则。** 需要写好 keep 保留规则，有些内容混淆之后可能会导致崩溃。

**核心一点是混淆了之后可能找不到数据**，一般的场景有：

1. 网络请求的实体类，混淆了可能找不到返回数据。
2. JNI 调用的 native 方法需要保留，防止在 C/C++ 层会找不到方法。
3. 用到反射的地方，反射需要字符串全类名，混淆了就找不到对象。
4. 第三方 SDK 的调用，别人的内部代码不能混淆。

---

### 应用 APK 怎么瘦身？

一个标准的 APK 通常由**代码、资源、底层 SO 库**三部分组成。

1. **在代码层面：** 开启代码混淆，对代码进行压缩剔除，可以节省一部分空间。
2. **在资源层面：** 搭配代码混淆开启 `shrinkResources`，这里是跟代码混淆绑定的，先检查代码是否可压缩，再跟着这些代码来判断资源是否有用。当然这里需要根据需求来，不能随意删减。
3. **最后在 SO 库层面：** 目前 99% 的真机都是 ARM 架构，可以过滤掉不必要的 X86 架构，精简很多空间。

如果还需要进一步瘦身，可以考虑把非核心的资源做成动态下发。

---

### 如何加载 Bitmap 并防止内存溢出？

超大的 Bitmap 在解码后会占用一整块完整内存，不注意的话很容易产生内存碎片，进而导致 OOM。

**防止 Bitmap OOM 的核心就是：** 降低图片尺寸、降低图片质量、避免重复创建。

**常见的优化方式主要有：**

1. **按需采样加载：** `inJustDecodeBounds` 获取图片尺寸，再计算 `inSampleSize`，避免加载原始大图。
2. **降低 Bitmap 的像素点精度：** 例如使用 `RGB_565` 代替 `ARGB_8888`，减少内存占用。
3. **避免请求重复创建或引用 Bitmap。** 在工作中遇见过桌面小卡片与应用之间通过 RemoteView 通信时，反复的传入更新 Bitmap 的指令，导致内存增长后卡片卡死无响应的现象。

---

### 如何对网络请求进行优化？

1. 通过本地缓存，如果不是强需求联网加载，数据没有变化就直接从缓存读取，减少请求次数。
2. 必须的网络请求，一般异步操作放到子线程执行。避免主线程阻塞导致界面卡顿。
3. 如果请求的数据量较大，比如大量资源的云同步，可以使用分批、分页加载。
4. 合理设置超时与重试：
   - 设置连接/读取超时，避免长时间卡死。
   - 对失败请求做指数退避重试，避免瞬间洪峰。

---

### 缓存一般怎么配置？

一般会采用**多级缓存的方式（Memory + Disk + Network）**的方式。

1. **优先从内存缓存读取：** 比如使用 LruCache 或者 Map。内存读取速度快，适合数据量小、访问频繁、生命周期和应用一致的数据。
2. **如果内存没有命中，再从磁盘读取：** 比如 MMKV、SQLite 这些。适合数据量较大、或者需要持久化的数据。即使重启仍然可以使用。
3. **如果网络缓存也没有命中，再去请求网络，再把结果写入缓存。**

---

### Android 应用冷启动的过程（包含 Context 和 View 绘制）？

**回答示例：**

冷启动就是应用进程不存在，用户点击桌面图标后系统要从零创建进程并启动 Activity。整体可以分成几个阶段：

**1. 启动请求发起**

用户点击 Launcher 图标，Launcher 进程通过 `Instrumentation.execStartActivity()` 把启动请求跨进程发给 system_server。这一步就是把"我要启动某个 Activity"的意图交给系统。

**2. 系统进程调度**

AMS/ATMS 接收请求，判断是否已有进程可复用。如果没有，就通过 Zygote fork 出一个新的应用进程。

**3. 应用进程初始化**

新进程进入 `ActivityThread.main()`，通过 Binder 回调 `attachApplication()` 把自己注册到 AMS。AMS 反向调用 `bindApplication()`，应用进程创建 Application 实例。

此时 Context 出现：

- 在 `handleBindApplication()` 中，系统会创建 LoadedApk，并基于它构造 ApplicationContext。
- 这个 Context 作为全局环境，提供资源访问、系统服务等能力。
- 随后在 `performLaunchActivity()` 中，Activity 实例被创建，并且会绑定一个 ActivityContext（其实是 ContextImpl 的实例），它和 ApplicationContext 不同，专门用于界面相关操作。

**4. Activity 启动与界面绘制**

- Activity 完成 `onCreate()`、`onResume()`。
- 在 `onResume()` 之后，Activity 调用 `makeVisible()`，把 DecorView 提交给 WindowManager。
- 系统创建 ViewRootImpl，作为界面渲染的入口。
- `ViewRootImpl.performTraversals()` 依次完成 `measure → layout → draw`，把整个 View 树测量、布局、绘制。
- 绘制结果提交到 Surface，由 SurfaceFlinger 合成，最终显示到屏幕。
- 首帧绘制完成后，系统替换掉占位窗口，用户看到真正的界面。

**总结一句话：**

冷启动就是：Launcher 发请求 → AMS 调度并 fork 新进程 → 应用进程初始化 Application 和 Context → Activity 创建并绑定 ActivityContext → Activity `onResume` 后触发 ViewRootImpl 绘制 → 首帧替换占位窗口。

---

### 如何优化 APP 启动过程的？

**从系统启动流程来理解：**

**1. 一个应用从点击图标到显示，大致流程是：**

1. Launcher 通过 Binder 调用 AMS，触发 `startActivity`
2. AMS 检查进程是否存在，不存在首先 Zygote fork 一个新进程
3. 新进程启动后进入主线程 `ActivityThread.main()`，创建主线程 Looper 并 attach 到 AMS
4. AMS 通过 Binder 调用 `bindApplication`
5. 主线程开始完成 Application 的创建和初始化
6. 执行 Activity 的生命周期 `create → start → resume`
7. 创建 ViewRootImpl 并执行 `measure → layout → draw`
8. 首帧提交到 SurfaceFlinger 渲染显示

**所以启动时间主要消耗在三部分：**

1. 进程创建
2. Application 初始化
3. 首帧的绘制

**优化的核心原则是：** 减少主线程路径任务 + 延迟非必要任务 + 增加并行度

**进程创建：**

如果从 ODM 或系统层来看，可以通过控制系统应用的启动策略，同时减少不必要的常驻进程降低负载。尽量让应用启动阶段能够调度到大核执行。三方 APP 这一阶段很难优化。

**Application 初始化：**

在 `bindApplication` 阶段，主线程会完成 Application 创建。如果在这里执行大量初始化逻辑，会直接阻塞主线程，延长应用启动时间。实际优化的是，减少启动关键路径上的工作量，比如延迟或者异步初始化，把部分资源延迟加载或者放到后台线程执行。三方 APP 无法定制 Android 框架代码，这里一般是通过内部构建启动任务调度框架来优化。

**首帧 UI 渲染：**

整体目标就是缩短点击到首帧出帧的时间，提升用户感知的启动速度。对首屏页面进行布局优化减少开销，或者复杂逻辑延迟到首帧后执行。分阶段加载内容，先展示核心 UI。

---

### 如何对 WebView 进行优化？

因为 WebView 是 chromium 内核，首次创建时需要完成较多初始化工作，比如：

- 加载底层库，初始化浏览器进程
- 初始化 JS 引擎，创建资源渲染环境

所以，第一次 `new WebView` 会有耗时问题。

**优化的核心一般是：** 预加载 WebView + WebView 池化复用 + 离线包资源加载。目的是减少首次初始化和网络加载带来的耗时。

**常见的优化方式有：**

1. **WebView 预加载或复用：** 提前创建 WebView，让内核提前完成初始化。维护一个 WebView 池，避免频繁创建销毁的开销。
2. **离线包加载：** 把完整资源提前下载到本地，减少网络请求时间。
3. **资源缓存或请求拦截：** 通过 `shouldInterceptRequest()` 对网络加载资源进行缓存或本地替换，减少加载时间。
