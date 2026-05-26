Java基础
C++和 java的区别
首先 C++ 追求的是性能和对底层的掌控权
Java 主要关注开发更快更安全，并且可以跨平台使用

从四个核心维度看：内存管理+指针+跨平台+继承
内存管理：
C++需要内存要自己申请，用完自己需要释放防止内存泄漏
Java 提供了 GC 机制来处理内存即可，GC可能会稍微影响业务，但减少了内存泄漏问题的出现

指针
C++允许直接用指针来操作内存地址，但如果使用不当可能直接把整体挂掉
Java 虽然底层也是地址，但表面做好了安全引用，防止内存使用错误

跨平台
C++直接把源代码处理成机器语言指令，因此在windows和linux这种跨平台之间，由于机器语言指令不同，无法跨平台使用
Java 额外引入了一层字节码，有了字节码就可以跨平台使用

类继承
C++一个类可以多重继承，允许有多个父类，可能会发生逻辑混乱
Java中规定一个类只能单继承，但额外加了接口可以多实现，作为能力的灵活性扩充

抽象类与接口的区别和共同点
- 抽象类：可以有成员变量、构造方法、普通方法；适合描述"is-a"关系。
- 接口：
只能定义方法签名，只能有 abstract方法，无方法体（Java 8+ 可有default， static 方法 可以有方法体），适合描述"can-do"能力。

共同点：都不能被实例化
👉 速答：抽象类是方法复用的模板，接口是定义一种类要实现的规范，作为灵活的能力补充。

---
接口中引入了 default 方法和 static 方法，请说明它们的区别和使用场景。
- default 方法
  - 接口中可以有带方法体的实现。
  - 实现类可以继承或选择性重写。
  - 主要用于接口升级时，避免破坏已有实现类。
  - 例子：List.sort() 就是通过 default 方法实现的。
- static 方法
  - 属于接口本身，不属于实现类。
  - 不能被继承或重写。
  - 必须通过 接口名.方法名() 调用。
  - 例子：Comparator.comparing() 就是接口里的静态方法。

"default 方法是给实现类的可选实现，可以继承和重写；
static 方法属于接口本身，不能继承，只能通过接口名调用。

default 解决接口升级问题的兼容性问题，static 提供工具方法。"
回答   "接口里的 default 方法就是给实现类一个'兜底方案'。比如接口升级加了新方法，如果没有 default，所有实现类都得改一遍，太麻烦了；有了 default，老代码不用动也能跑。它相当于接口自己带了个默认实现，类可以直接用，也可以自己改掉。"

---
多态怎么理解
多态本身是面向对象的三大特性之一，
同一个方法，在不同对象上有不同表现。
两种形式：
- 编译时多态（静态多态）：通过方法重载实现，编译阶段决定调用哪个方法。
- 运行时多态（动态多态）：通过方法重写实现，运行时根据对象的实际类型决定调用哪个方法。编译时看左边（声明类型），运行时看右边（实际对象类型）

常见的使用就是
List<Integer> res = new ArrayList<>()
所谓编译看左边，运行看右边

---
有了基本类型为什么要包装类？自动装拆箱是，频发装箱有啥影响？
首先Java是一个面向对象的语言，但为了保证性能，提供了8个基础数据类型，
他们不是对象，没有继承object，因此也没有额外的方法调用。在一些场景下不可用，因此需要包装类型

1. 最常见的集合类
由于java编译器会执行泛型擦除，编译时候会擦除为object类，比如List<Integer>,由于int根本不是object类，这里如果写int直接编译报错
2. 空值的表达
基本数据类型是有默认值的，比如int是0，就无法在业务中准确判断是空值，还是默认值。
而Integer是可以通过null表达空状态的，防止表达却是
3. 功能性
封装之后有更多方法，灵活性更好，比如字符串转数字，就可以用Integer.parseInt

自动装拆箱，属于jvm提供的一种快捷使用
装箱：int——integer  编译器会在需要时自动插入 valueOf()（装箱）
拆箱：integer——int xxxValue()（拆箱）方法。

对象创建开销
每次装箱都会创建新的包装类对象（除非命中缓存池，如 Integer.valueOf() 对 -128~127 范围的缓存）。
频繁创建对象会增加 GC 压力，可能就会出现卡顿现象m

潜在的空指针问题
拆箱时如果包装类对象是 null，会抛出 NullPointerException。

---
Integer的缓存池机制是如何工作的？范围是多少？能否调整？
缓存池：
Integer.valueOf(int i) 方法会优先从缓存池中返回对象，而不是每次都新建。
范围：
默认缓存范围是 -128 ~ 127。
实现原理：
在 Integer 类内部有一个静态内部类 IntegerCache，
在类加载时就会预先创建好这个范围内的 Integer 对象，并缓存到数组里。
private static class IntegerCache {
    static final Integer cache[] = new Integer[-(-128) + 127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Integer(i - 128);
    }
}
可以通过 JVM 参数 -XX:AutoBoxCacheMax 调整上限，但上限只能大于127，只能大不能小但
下限固定为 -128

---
final、static、synchronized
- final：修饰类（不可继承）、方法（不可重写）、变量（不可修改）。
- static：修饰的变量或者方法属于类本身，不依赖实例。可以修饰内部类，顶层不行。
- synchronized：保证线程安全，方法或代码块加锁。

Final 表示一种不可变的状态！
static 表示变量或者方法的归属属于定义的类或接口本身，先于实例对象存在的，因此不能用this super
单 static 修饰是可以被修改的

👉 速答：final 限制、static 属于类、synchronized 保证并发安全。

---
String、StringBuffer、StringBuilder
- String：不可变，线程安全。
- StringBuffer：可变，线程安全（加锁）。
底层是一个可变的 char[]，带有容量扩展机制。
内部方法大多用 synchronized 修饰，保证多线程安全。
修改时直接在原数组上操作，不会频繁创建新String对象。
- StringBuilder：可变，非线程安全，效率高。
底层同样是可变的 char[]，扩展机制和 StringBuffer 一样。就是不加锁，单线程效率高！

👉 速答：String 不可变，Buffer 安全，Builder 快。

---
String 不可变的原因
底层是一个 final char[]，不可变。
每次修改都会生成新的对象，原数组不变。
好处：不可变性保证了线程安全，适合作为常量池、哈希表的 key。
- 安全性：避免被篡改（如网络地址、类加载）。
- 性能：字符串常量池复用。
- 线程安全：不可变对象天然安全。

👉 速答：String 不可变是为了安全、性能和线程安全。

---
new String("ABC")创建了几个对象？
主要差异需要看  字符串常量池 是否包含了"ABC" 这个对象

---
equals 与 ==、hashCode
- ==：比较引用地址是否相同。如果是基本数据类型相当于直接比较数值
- equals：比较内容是否相同（可重写）
- hashCode：对象的哈希值，用于集合（HashMap、HashSet）。

如何重写equals()方法？为什么还要重写hashCode()
之所以重写 equals，原因主要是因为object 类 提供的 equals 方法，比较的是两个对象之间的引用地址。

但实际业务中，我们更关心两个对象之间的属性值，是否相等。

因此，重写equals一般发生在自定义一个类对象后，判断该类的实例对象是否间是否相等的关系

具体实现："重写 equals() 的过程中要先判断是否同一个对象，再判断是否属于同一个类，最后逐一比较关键字段。
其中前两步相当于性能优化，快速帮助比较。
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Person person = (Person) o;
    return age == person.age &&
           Objects.equals(name, person.name);
}

如果属性字段比较多，最终的整体逻辑就会比较重。

因此在lava的一些集合类的实现中，比较两个对象时，通常都会先根据传入对象的hashCode比较，
但是在 java 中，如果两个对象相等，那么他们的hashCode就一定相同。
而反过来如果两个对象的hashcode相同，也可能只是"哈希碰撞"

所以需要注意，如果我们重写了equals0方法，那么就要注意hashCode方法，一定要保证能遵守上述规则。

重写 hashCode() 方法的核心就是：保证和 equals() 一致，同时尽量减少哈希冲突。
📌 重写原则
1. 一致性：如果两个对象 equals() 返回 true，它们的 hashCode() 必须相等。
2. 稳定性：对象在生命周期中，只要关键字段不变，hashCode() 返回值必须一致。
3. 高效性：哈希值分布要尽量均匀，减少冲突。
常见写法
- 推荐用 JDK 提供的工具类：Objects.hash，把对象的字段传入即可
@Override
public int hashCode() {
    return Objects.hash(name, age); // 自动组合字段生成哈希
}

👉 面试速答：== 比地址，equals 比内容，hashCode 保证集合查找。
👉 面试速答：重写 equals 必须重写 hashCode，否则集合行为异常。

---
深拷贝与浅拷贝
- 浅拷贝：复制对象引用，指向同一内存。
- 深拷贝：复制对象内容，生成独立副本。

👉 面试速答：浅拷贝共享引用，深拷贝独立对象。

Object 的 clone 在底层是一个native 原生的方法，调用的时候就在内存开一块新空间。
把老对象的数据每一位复制过去。

这时候，如果复制的是基本数据类型，相当于直接复制了数值
但如果是引用数据类型，相当于复制了一份地址，
此时相当于新旧对共用同一块数据，没有做到拷贝的独立

因此要实现深拷贝的方法：
1. 嵌套式的重写，每个引用字段都克隆一遍，不推荐
2. 利用序列化和反序列化来拷贝，相当于把对象拆开重新再构建一个

---
Error 与 Exception
- Error：严重错误，程序无法恢复（如 OutOfMemory。StackOverflow）。
- Exception：可捕获处理的异常。
编译时异常（Checked Exception）
- 编译器强制要求处理（try-catch 或 throws）。
- 例如：IOException、SQLException。
- 特点：程序可预期，必须显式处理。
运行时异常（Unchecked Exception）
- 继承自 RuntimeException。
- 编译器不强制处理。
- 例如：NullPointerException、ArrayIndexOutOfBoundsException。
- 特点：通常由程序逻辑错误引起。

👉 面试速答：Error 不可控，Exception 可处理。

---
反射机制？它会带来哪些性能开销？如何优化？
- 定义：运行时动态获取类信息、调用方法、访问属性。
- 应用场景：框架（Spring、Hibernate）、序列化、动态代理。

获取 Class 对象的三种方式
1. Class.forName("类全名")
  - 传入类的全限定名（包名+类名），返回对应的 Class 对象。
  - 常用于框架加载配置里写死的类名，比如 JDBC 驱动：
Class.forName("com.mysql.cj.jdbc.Driver");
2. 对象.getClass()
  - 通过已有对象实例获取其运行时的 Class 对象。
String s = "hello";
Class<?> clazz = s.getClass();
3. 类名.class
  - 直接通过类字面量获取 Class 对象。
  - 编译期就能确定，常用于泛型、注解等场景：
Class<String> clazz = String.class;

性能瓶颈
1. 寻找的过程慢，需要字符串类名在内存查找；jvm需要进行较多的校验
2. 反射可以访问和修改类的字段方法，可能造成安全问题

优化
1. 加缓存，避免每次使用都要搜索
2. setAccessible(true) 可以让 jvm 避开一些权限检查

👉 面试速答：反射是运行时操作类，常用于框架和动态代理。

---
静态代理与动态代理的区别是什么？JDK动态代理与CGLIB动态代理有何不同
- 静态代理
  - 在编译期就确定代理类，代理类需要手写或提前生成。
  - 优点：结构清晰，易于理解。
  - 缺点：每个接口都要写一个代理类，代码冗余，扩展性差。
- 动态代理
  - 在运行时通过反射或字节码生成代理对象。
  - 优点：无需为每个接口写代理类，灵活、可复用。
  - 缺点：实现复杂，性能略低于静态代理。

👉 面试速答：
"静态代理在编译期就确定代理类，代码冗余；动态代理在运行时生成代理对象，更灵活，常用于 AOP、RPC 等场景。"

JDK 动态代理 vs CGLIB 动态代理
暂时无法在飞书文档外展示此内容

---
泛型类型擦除并说说其局限性？为什么不能创建泛型数组，
泛型在编译期的作用
- 编译器会检查类型安全：比如 List<String> 只能放 String，放别的会报错。
- 这些信息只存在于 编译阶段，保证代码写的时候不出错。

类型擦除的含义
- 到了 运行时，JVM 并不保留泛型的具体类型信息。
- 所有泛型参数都会被擦除成它们的 上界（默认是 Object）。
List<String> list1 = new ArrayList<>();
List<Integer> list2 = new ArrayList<>();
System.out.println(list1.getClass() == list2.getClass()); // true
- 运行时它们的类型都是 ArrayList，没有区别。

- 定义：编译时保留泛型，运行时擦除为 Object。
- 局限性：
Java 泛型在编译期有效，运行时类型信息会被擦除成 Object 或上界。这样保证了向后兼容，
但局限是jvm运行时不能直接获取泛型类型、反射也拿不到泛型参数

不能创建泛型数组，
类型擦除本身是一个折中的方案，由于java诞生较早，而泛型是后引进的，
因此为了使老版本代码运行，设定了jvm运行时不去关注类型

而不能创建泛型数组，原因是一个根本的逻辑冲突
数组有运行时类型检查，在运行的时候需要明确存储类型
而 泛型被擦除，运行时信息类型缺失，使用泛型数组可能会导致程序crash

👉 面试速答：泛型运行时被擦除，限制了类型操作。

---
Lambda表达式的底层实现原理是什么
核心是
编译器到运行期的能力交接

编译器遇到 Lambda 表达式时，不会直接生成匿名内部类，而是生成一条 invokedynamic 字节码指令。
这条指令相当于一个"钩子"，告诉 JVM：运行时需要动态决定如何执行这个 Lambda。

当 JVM 执行到这条 invokedynamic 指令时，会调用 LambdaMetafactory 工厂方法。
动态生成一个函数式接口的实现类（通常是通过字节码生成）。
最终返回一个目标方法的引用，完成 Lambda 的运行。

好处是：
1. 运行时动态生成对象，不会产生额外的类文件。比每个类单独一个类文件加载开销低！
2. JVM 可以不断优化字节码指令，而我们的代码不用改，就能直接享受优化。

1. 写 () -> {} Lambda表达式
2. 编译期：生成invokedynamic字节码指令
3. 运行期：调用LambdaFactory
4. 工厂动态生成实现类 + 创建实例
5. 绑定方法引用/代码体，完成调用

---
Java的注解是什么
类似于 商品的标签和扫码枪
谁是拿扫码枪的很重要！

1. 注解是一种元数据，本质上是一个没有逻辑的标签
2. 注解的作用主要却决于读取他的对象
比如编译器用 @Override 做语法检查，JUnit 用 @Test 做测试识别。
注解本身只是标签，逻辑在读取者。

---
Java中异常分哪两类？有什么区别？
受检和非受检
受检异常是由于环境不确定导致的意外，比如文件不存在、网络中断。编译器会审查，并要求提前准备好降级方案（try catch捕获）
非受检异常是逻辑不严密导致的低级错误，比如空指针数组越界，靠代码审查和测试来消灭它。

---
Java中try_catch_finally中的finally代码块一定会执行吗
不一定，在 jvm 正常运行的前提下
如果 jvm 因为各种原因挂掉，或者手动调用了 jvm 终止，无法在正常执行finally了

---
Java集合
谈谈 List、Set、Map 的区别
List：有序可重复，可以按下标取值。可能会有多个空值存在，所以为防止空指针，有时候需要额外判断。
Set：无序不可重复，用来去重。允许存在一个null
Map：键值对存储，key 唯一，value 可重复。要看实现，HashMap 可以有一个 null key，Hashtable/ConcurrentHashMap 不行。

👉 大白话：List 像队伍，Set 像集合去重，Map 像字典。

---
谈谈 ArrayList 和 LinkedList 的区别
ArrayList：底层数组，查找快，插入删慢。
LinkedList：底层是一个双向链表，插入删快，查找慢。

👉 大白话：ArrayList 查找快，LinkedList 插入删快。

---
请说一下 HashMap 与 HashTable 的区别
HashMap：非线程安全，效率高，允许 null。
Hashtable：线程安全（方法加锁），效率低，不允许 null。
效率低：所有方法都加了 synchronized，在单线程或低并发场景下性能差。
ConcurrentHashMap 在并发场景下更高效。

👉 大白话：HashMap 更常用，Hashtable 已过时。

---
谈一谈 ArrayList 的扩容机制？
底层就是一个 Object[] elementData 数组。
默认构造函数里并不会立刻分配数组，而是在第一次 add 时才分配默认容量 10：
private static final int DEFAULT_CAPACITY = 10;
当元素数量超过当前数组容量时，就会触发扩容。
默认容量 10，满了就扩容。
扩容规则：新容量 = 旧容量 * 约1.5。

本质：新建数组，把旧数据拷过去。

👉 大白话：ArrayList 扩容就是数组拷贝，按 1.5 倍扩。

为什么不用按需扩容呢？倍数扩容不会有内存浪费吗？
"ArrayList 不采用按需扩容，因为每次扩容都要复制数组，按需扩容会导致整体插入性能退化到 O(n²)。

它采用 1.5 倍扩容，是在减少扩容次数（性能）和避免过度浪费内存之间的折中方案。

同时 ArrayList 有懒加载机制，不会一开始就占用内存，只有第一次 add 才分配默认容量 10。
但是为了尽量避免扩容带来的消耗，如果一开始就能给出list的具体容量最好"

---
CopyOnWriteArrayList的实现原理及适用场景。为什么它是"弱一致性"的？
底层结构：内部维护一个 volatile 的数组 Object[] array
读操作（get/遍历）：
- 无锁
- 直接读取当前数组引用
- 因为数组是不可变快照，所以读永远安全
写操作（add/remove/set）：
- 加 ReentrantLock 独占锁
- 复制当前数组 → 修改副本 → 新副本替换原数组引用

弱一致性是指，他这种机制只能保证最终的一致性，但不能保证实时的读取和修改一致。
体现在当使用迭代器进行遍历的时候，迭代器会保存一个读到数组的快照。而不一定是正在被修改的最新的数组
这是为了高并发性能的一种妥协方案。

因此，他这种写时复制的机制，更适合于读多写少的场景。
因为大量的写操作引发的加锁——复制——替换引用本身对内存就是一种消耗

---
HashMap 的实现原理？可能出现的问题是?
底层：数组 + 链表/红黑树。1.8以后引入
插入：先算 hashCode 定桶，再用 equals 定 key。

查找：同样先定位桶，再 equals 精确匹配。

JDK8 优化：
链表超过阈值转红黑树。1.8引入
在 JDK8 的 HashMap 里，链表转红黑树的 阈值是 8。
- 当某个桶里的链表长度 超过 8 时，会尝试把链表转成红黑树，以提升查找效率。
HashMap 在 JDK8 里，链表长度超过 8 就转红黑树，这个数字是性能权衡的结果：小于 8 用链表更快，超过 8 用树更合适。
源码里也写了树节点比链表节点大，所以只有在链表够长时才值得转树。再加上概率分布分析，超过 8 的情况本来就很少，树化只是防止极端碰撞。
- 但还有一个前提：整个数组长度要达到 64 以上才会真正转树，
否则先扩容数组，由于哈希运算依赖于数组长度，扩容可以让元素分散，降低碰撞概率。
- 反过来，如果树节点数量减少到 6 以下，会退回链表结构。

👉 大白话：HashMap 用 hashCode 定桶，equals 定 key。

扩容时可能发生死循环，1.8修改
📊 具体表现
当容量不足时，会进行扩容，把旧数组的元素重新分配到新数组。默认 capacity=16 loadFactor=0.75
- 在 JDK 7 的 HashMap 中，链表迁移时采用 头插法。每次迁移一个节点时，都把它插入到新桶链表的 头部。
- 多线程同时迁移时，可能导致链表节点指针错乱，形成 环形链表。
- 一旦形成环形链表，后续调用 get() 或迭代时，就会陷入 死循环，CPU 飙高。

📌 JDK 8 的改进
- JDK 8 改为 尾插法迁移，避免了环形链表问题。
- 同时引入了 红黑树结构，优化了高冲突场景下的性能。
- 但注意：HashMap 本身仍然 不是线程安全的，在并发场景下依然可能出现数据丢失或覆盖，只是不会再出现死循环。

---
请简述 LinkedHashMap 的工作原理和使用方式
在 HashMap 基础上，在每个节点内部就嵌入了双向链表指针，把所有节点串联起来。

static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after; // 双向链表指针
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
可以看到：
- 它继承了 HashMap.Node，所以仍然有 key, value, next（用于哈希桶链表）。
- 额外增加了 before 和 after 指针，用于维护插入或访问顺序的双向链表。
- 也就是说：每个节点同时存在于两条链上：
  - 一条是 HashMap 的桶链（用于哈希查找）。
  - 一条是 LinkedHashMap 的双向链表（用于顺序维护）。

额外维护了一条 双向链表 来记录元素的顺序。
          ┌───────────────────────────────┐
          │       Hash Table (数组)       │
          │  [0] [1] [2] [3] [4] [5] ... │
          └───────────────────────────────┘
                     │     │     │
                     ▼     ▼     ▼
                 ┌────┐ ┌────┐ ┌────┐
                 │Key1│ │Key2│ │Key3│  ← 每个桶指向一个节点
                 └────┘ └────┘ └────┘
                     │     │     │
                     ▼     ▼     ▼
        ┌──────────────────────────────────────────────┐
        │         双向链表（维护插入或访问顺序）        │
        └──────────────────────────────────────────────┘
        Head → [Key1,Value1] ⇄ [Key2,Value2] ⇄ [Key3,Value3] → Tail

常用场景：缓存，比如 LRU。

👉 大白话：LinkedHashMap 就是 HashMap + 链表，用来保持顺序，做缓存很方便。

---
谈谈对于 ConcurrentHashMap 的理解
数组+链表/红黑树

背景：为什么要有它
- HashMap 在多线程下不安全，Hashtable 虽然安全但效率太低（整张表加锁）。
- 所以出现了 ConcurrentHashMap：既要线程安全，又要高并发性能。

---
JDK7 实现：分段锁（Segment）
- 整个 Map 被分成若干段（Segment），每段都是一个小 HashMap。
- 每个 Segment 有自己的锁，线程操作不同段时可以并行。
- 优点：锁粒度比 Hashtable 小很多。
- 缺点：结构复杂，Segment 数量固定，扩容不灵活。

---
JDK8 实现：CAS + synchronized
在 JDK8 的 ConcurrentHashMap 里，插入流程是这样的：
1. 定位桶：通过 hashCode 算出数组下标，找到对应的桶（数组的一个槽位）。
2. CAS 插入头节点：如果这个桶是空的（null），就用 CAS（Compare-And-Swap）直接把新节点放到桶的头节点位置。这样不用加锁，效率很高。
3. 冲突时加锁：如果桶里已经有节点（链表或红黑树），CAS 就失败了，这时才会用 synchronized 锁住这个桶的头节点，保证只有一个线程能安全地修改这个桶里的结构。
4. 后续操作：在锁保护下插入新节点，或者在红黑树里做平衡调整。

---
性能与安全性
- 读操作几乎无锁，写操作锁粒度极细。
- 不会出现死锁，也不会抛出 ConcurrentModificationException。
- 适合高并发场景，比如缓存、统计计数、线程安全的共享数据结构。

---
👉 大白话速答版：
"ConcurrentHashMap 是高并发下的安全 Map。JDK7 用分段锁，每段独立加锁；JDK8 改成 CAS + synchronized，锁更细，读几乎无锁，效率更高。它就是在多线程里既安全又快的 HashMap。"

ConcurrentHashMap 的扩容和 HashMap 相比呢
- HashMap：扩容阈值是 capacity * loadFactor（默认 capacity=16 loadFactor=0.75），超过就扩容一倍数组。
- ConcurrentHashMap：同样默认 capacity=16 loadFactor=0.75，但实现更复杂：
  - 它在 多线程环境下，扩容不是单线程完成，而是多个线程协作。
  - 每个线程会分配一段桶区间来搬迁数据，避免全表阻塞。
  - 阈值计算和 HashMap 类似，但扩容过程更精细，保证并发安全。
  -

同时，如果正在扩容，会在原数组的桶位置加一个转发节点，是一个哈希值为负数的节点。
- 此时，如果另一个线程读取到这个节点，会先放弃加入，协助扩容

关键区别
- HashMap：单线程扩容，期间可能阻塞，读写都受影响。
- ConcurrentHashMap：多线程协作扩容，读操作基本不受影响，写操作只锁桶头。
- 阈值本质：两者都是基于 capacity * loadFactor，只是 ConcurrentHashMap 在并发场景下做了更复杂的控制。

---
为什么ConcurrentHashMap不允许Key或Value为null？
核心是为了消灭二义性，为了保证结果的唯一性和操作的原子性
如果允许 null，那么 map.get(key) 返回 null 时，就无法区分是 key 不存在 还是 key 存在但 value 为 null。
CHM 提供了 putIfAbsent、computeIfAbsent 等原子操作。
如果允许 null，这些方法的判断条件就会失效（因为无法区分"没值"和"值就是 null"

---
ConcurrentHashMap的size()方法是如何保证高并发下的统计准确性的？
核心是用一点的准确性差异，换取高并发下的性能提升。
具体是维护一个常规数组和临时数组
基础数组（baseCount）：默认修改目标，低并发时直接更新。
计数数组（CounterCells）：高并发下的分散槽位，线程更新失败时退到这里。随机放一下。
统计结果：size() 就是 baseCount + CounterCells 求和。不是实时严格准确的，用近似值换取高并发性能。

---
ConcurrentHashMap 能保证完全线程安全吗？
ConcurrentHashMap 的单个操作是线程安全的，
但复合操作（如 get+put）不是原子性的，可能在并发下出问题。

解决方法是用 putIfAbsent、computeIfAbsent 等原子方法。

---
ArrayBlockingQueue与LinkedBlockingQueue有何区别？怎么选
如果追求高执行效率，且对内存抖动敏感。选array。高性能是 固定容量 + 内存稳定 + 操作开销小（单锁，出入队共用）
如果有高并发的读写需求，能接受一点内存开销，选link。
总的来说link更适合标准的 生产者-消费者 模式

---
PriorityQueue的底层数据结构是什么？它是如何维持堆属性的？
数据结构上，本质上是维护了一个动态数组
处理逻辑上，利用数组的位置维护了一个完全二叉树。

对于每个节点，将他的下标i计算2i+1/2i+2，即为他的两个孩子节点。

维持堆属性方面
默认的java实现是一个小顶堆，把最小的元素放在首位

入队时，首先将新元素放在队尾，然后不断与他的父节点比较大小，直到确定位置。
出队时，直接将队首，即为堆顶元素出队。然后把队尾的元素填补到队首，再逐渐比较下沉。

可以理解为，入队是一个逐级提拔的过程，出队是一个关系户被逐级开除的过程！

---
DelayQueue的实现原理及其在定时任务中的应用

数据结构与排序逻辑
- DelayQueue 本质上是一个 基于 PriorityQueue 的无界阻塞队列。
- 队列里的元素必须实现 Delayed 接口，包含一个过期时间。
- 排序规则：按照过期时间升序排列，最早过期的任务在队首。

📖 Leader-Follower 模式
- 取任务的线程：只有一个线程会被 DQ 选为 leader，根据队首元素的过期时间设置定时。
- 其他线程：全部进入等待状态（睡眠），避免无意义的轮询。
- 好处：降低 CPU 消耗，避免多个线程同时竞争。

📊 应用场景
- 订单超时自动取消：
  - 把订单任务放入 DelayQueue，设置过期时间为"下单时间 + 超时时间"。
  - 后台设置一个线程，等着到期后自动触发取消逻辑。
- 定时任务调度：
  - 比如缓存过期清理、延迟消息队列。
- 避免轮询压力：
  - 如果用普通队列，需要不断检查是否过期，系统压力大。
  - DelayQueue 通过 leader-follower 模式，只在leader结束后，接力唤醒下一个

---
TreeMap和TreeSet的原理是怎样的
1. 红黑树的数据结构：染色+旋转，保证了左右子树的高度不会差别过大
首先二叉搜索树会有一个问题，如果不对元素加以限制，可能退化成一条链表，
比如持续输入 1-2-3-4-5，会退化成一条只有右子树的链表，失去了树的搜索优势
   (30,"C")(B)
      /        \
 (20,"A")(R)   (40,"D")(R)
   /               \
(10,"X")(B)       (50,"Z")(B)

2. TreeSet本质就是套皮了一层 Treemap
把存放的数据作为key，定义一个占位对象作为value
这样利用key不能重复，不能为空的机制，就可以实现了 TreeSet 的功能

---
Java多线程
为什么要使用多线程
1. 线程是cpu调度执行的最小单位，管理成本低于进程
2. 现代系统中业务中存在大量并发，多线程可以提高可用性

---
线程安全怎么理解？有哪些实现方式吗
多个线程同时访问某个资源时，能够保证数据的一致性和正确性

加锁同步：保证资源只能被一个线程访问
原子变量：保证对变量的操作是原子的
不可变对象：不允许修改肯定是线程安全的

---
Volatile 关键词的作用是？
Volatile 是用来修饰变量的一个关键词，它的作用是
1. 保证变量的可见性，表明这个变量是共享的，每次访问的时候需要到主存读取
2. 防止编译器优化的这种指令重排序，读取变量的时候会插入特定的内存屏障，保证前后指令的特定顺序。

---
Java 中使用多线程的方式有哪些？/ 启动一个线程的方式有哪些

启动线程的方式主要有四种：
继承 Thread 类、Thread 类本身就实现了 Runnable
实现 Runnable 接口、
实现 Callable 接口配合 FutureTask、futuretask本质上也是一个实现runnable接口的类，用来接收结果
以及使用线程池。

它们本质上最终都是调用 Thread.start() 来启动线程，只是任务的封装方式不同。

---
请谈谈 Thread 中 run() 与 start() 的区别？

run() 方法最初定义在 Runnable 接口里，表示线程要执行的任务逻辑。

Thread 类实现了 Runnable，因此也有 run()。即使不传入接口 Runnable 实现对象也可以运行。

如果你又选择继承了 Thread 并重写 run()，那就会执行你写的 run 逻辑；

如果是实现 Runnable 接口后传给 Thread，那么调用 Thread.run() 为执行实现 Runnable 后重写的run()。

Thread.run() 方法
@Override
public void run() {
    if (target != null) {
        target.run();   // 如果传入了 Runnable，就执行它的 run()
    }
    // 如果 target == null，就什么都不做
}
- 情况 1：传入 Runnable → 调用 target.run()，执行你实现的任务逻辑。
- 情况 2：没传 Runnable → 默认空实现，什么都不做。
- 情况 3：继承 Thread 并重写 run() → 覆盖掉这个方法，执行你写的逻辑。

而 start() 才是启动新线程，最终由 JVM 调度执行 run()

---
线程池的状态有哪些？
暂时无法在飞书文档外展示此内容

1. RUNNING → SHUTDOWN：调用 shutdown()。
2. RUNNING/SHUTDOWN → STOP：调用 shutdownNow()。
3. SHUTDOWN → TIDYING：队列任务执行完毕，线程池大小为 0。
4. STOP → TIDYING：所有任务终止，线程池大小为 0。
5. TIDYING → TERMINATED：调用 terminated() 方法，线程池彻底结束。

---

---
为什么不推荐使用 Executors.newFixedThreadPool()

问题核心
- Executors.newFixedThreadPool() 内部使用 无界队列 LinkedBlockingQueue（容量是 Integer.MAX_VALUE）。
- 当任务提交速度 > 线程处理速度时，队列会无限堆积，最终可能导致 内存溢出（OOM）。
- 线程池参数被隐藏，开发者无法灵活配置核心线程数、最大线程数、队列大小和拒绝策略。

具体风险
1. 内存风险：无界队列无限增长，可能撑爆内存。
2. 扩展性差：线程数固定，无法根据负载动态扩容。
3. 拒绝策略不可控：默认 AbortPolicy，直接抛异常，不利于业务容错。
4. 参数不可见：工厂方法封装了 ThreadPoolExecutor，开发者不了解底层配置，难以调优。

推荐做法
直接使用 ThreadPoolExecutor，手动配置参数：
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                      // 核心线程数
    4,                      // 最大线程数
    60, TimeUnit.SECONDS,   // 空闲线程存活时间
    new ArrayBlockingQueue<>(100), // 有界队列，避免无限堆积
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.AbortPolicy() // 拒绝策略，可根据业务调整
);

不推荐使用 Executors.newFixedThreadPool()，因为它内部使用无界队列，任务过多时可能导致 OOM。推荐直接用 ThreadPoolExecutor

---
说一下线程的几种状态？
速答版
"Java 线程有六种状态：NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED。RUNNABLE 包含就绪和运行，
BLOCKED 是等锁，
WAITING/TIMED_WAITING 是等待，
最后 TERMINATED 表示结束。"

- NEW
  - 新建状态，线程对象已创建但还没调用 start()。
- RUNNABLE
  - 就绪/运行状态，线程已调用 start()，等待 CPU 调度或正在运行。
  - 注意：Java 把"就绪"和"运行"统一叫 RUNNABLE。
- BLOCKED
  - 阻塞状态，线程在等待获取锁（synchronized）。
- WAITING
  - 无限期等待状态，调用 Object.wait() 或 join() 等方法，直到被唤醒。
  - wait() 会让线程进入 WAITING，需要其他线程 notify() 才能唤醒。
- TIMED_WAITING
  - 有时限的等待状态，比如 sleep(ms)、wait(ms)、join(ms)。
  - sleep() 会让线程进入 TIMED_WAITING，时间到自动恢复
- TERMINATED
  - 终止状态，线程执行完毕或异常退出。
NEW --> RUNNABLE --> TERMINATED
          |
          |---> BLOCKED (等待锁)
          |---> WAITING (无限等待)
          |---> TIMED_WAITING (限时等待)

说下

谈一谈线程sleep()和wait()的区别？
好，我们借着线程状态这一题，把 sleep() 和 wait() 的区别整理一下：

---
线程 sleep() vs wait()
相同点
- 都会让当前线程进入等待/暂停状态。
- 都可能导致线程暂时不执行。
区别点
暂时无法在飞书文档外展示此内容
速答版
"sleep() 是 Thread 的静态方法，让线程休眠一段时间，不释放锁，时间到自动恢复；
wait() 是 Object 的方法，必须在同步块里调用，会释放锁，进入等待队列，需要其他线程 notify() 才能唤醒。

就能把 状态题 和 方法区别题串起来：
- 状态题：线程可能进入 TIMED_WAITING 或 WAITING
- 区别题：sleep 不释放锁，wait 释放锁并依赖 notify。

java线程中notify 和 notifyAll有什么区别？
notify() 唤醒一个等待线程进入 RUNNABLE，具体哪个由 JVM 决定；
notifyAll() 唤醒所有等待线程，让它们重新竞争锁。

前者效率高但不公平，后者公平但可能带来性能开销。

那正好说说如何实现多线程中的同步？

最基本的是 JVM 关键字 synchronized，
更灵活的是 Lock；
轻量操作可以用基于 CAS 的原子类；无锁实现性能高
线程协作可以用并发工具类；CountDownLatch, CyclicBarrier, Semaphore

并发工具类的本质都是基于 AQS，而 AQS 的核心就是 CAS + 队列。
CountDownLatch 用 CAS 递减计数，Semaphore 用 CAS 更新permit，CyclicBarrier 用锁和条件变量配合 CAS。可以说它们的底层都是靠 CAS 保证原子性。"

变量可见性用 volatile；

那 说说 你对 synchronized 的了解吧
- 作用：Java 提供的同步机制，保证同一时刻只有一个线程能访问被锁的资源。
可以用来处理方法、代码块。本质上是对对象上锁。是通过不同位置修饰来实现给不同对象上锁

📌 使用方式
1. 修饰方法 → 实例方法锁当前对象实例。静态方法锁当前类的 Class 对象，所有实例共享。
2. 修饰代码块 → 括号内锁指定对象或类Class对象。

👉 总结：synchronized 加在实例方法上锁对象实例，加在静态方法或 synchronized(Class) 上锁类对象。

本质上就是基于 对象头 (Object Header) 里的锁标记来实现的。
具体机制
- 对象头结构：每个 Java 对象在内存里都有一个对象头 (Object Header)，其中包含 Mark Word，里面存储了标识对象的哈希码、GC 信息、以及锁状态。

📌 特殊点
- 构造方法不能用 synchronized 修饰，但可以在内部写同步代码块。

面试速答版
"synchronized 是 Java 的同步关键字，保证资源在同一时刻只被一个线程访问。它可以修饰实例方法（锁对象）、静态方法（锁类）、代码块（锁指定对象）。

这个synchronized的原理是什么

底层原理
都是在字节码层面
- 同步代码块：编译成 monitorenter / monitorexit 指令。
- 同步方法：方法表里有 ACC_SYNCHRONIZED 标识。调用时自动加锁

- 最终实现：都依赖对象头markword里的锁标记来记录当前锁的不同状态
[图片]
对象头里的 Mark Word 只是用来存储锁状态和指向 Monitor 的信息。

👉 速答版：
"

---
介绍下 Synchronized 的锁升级吧
synchronized 的锁会根据 竞争情况逐步升级，目的是在保证线程安全的同时尽量提高性能。
偏向锁（Biased Lock）
- 特点：Mark Word 记录第一个获得锁的线程 ID。
- 优势：同一线程反复进入同步块时，无需真正加锁，性能最好。
- 触发条件：无竞争，单线程反复进入。
轻量级锁（Lightweight Lock）
- 特点：Mark Word 指向线程栈中的 Lock Record。
- 机制：线程通过 CAS 尝试获取锁，如果失败会自旋等待。
- 触发条件：有多线程竞争，但冲突不严重。
重量级锁（Heavyweight Lock）
- 特点：Mark Word 存放一个指针，指向 JVM 内部的 Monitor 对象。
- 机制：线程挂起，进入 Monitor 的等待队列，由操作系统调度。
- 触发条件：竞争激烈，自旋失败。

---
谈谈线程死锁，如何有效的避免线程死锁？
首先线程死锁是指多个线程同时被阻塞了，并且他们一个或多个都在等待某个资源的释放，
形成了循环等待，导致无限期阻塞。

产生死锁的四个必要条件是：
资源互斥
线程已经持有资源不释放，同时又在请求其他资源。
其他线程正在使用的资源不可被剥夺
多个线程形成了循环等待。

只要破坏一个条件即可避免

---
谈谈线程阻塞的原因？
线程阻塞的原因主要有：

等待锁、等待条件变量、等待其他线程、
I/O 阻塞，
以及死锁。比如 sleep 不释放锁，wait 会释放锁。
join 等待其他线程结束，yield 只是让出 CPU

---
synchronized和volatile关键字的区别？
首先，作用的对象来说，
Volatile 只能修饰变量，synchronized 可以修饰方法，代码块。

另一方面， Java 内存模型（JMM）描述在并发编程时对共享变量的要求要保证：
原子性、有序性、一致性

而 volatile 关键字，
首先可以保证有序性，修饰的变量在读写时会在字节码层面加入内存屏障，防止了重排序
其次可以保证一致性，jvm每次读取变量都需要到主存中读取。
但是，不能保证原子性！比如对一个变量的修改，需要通过 读取-操作-写回 这三个步骤，
volatile无法保证这个流程的原子性。

而 synchronized 都可以保证，通过加锁保证了方法块内代码的原子性
并且，在进入退出修饰的方法或者代码块的时候，会把本地的缓存数据和主内存中刷新。保证了一致性和可见性。

---
谈谈ThreadLocal用法和原理？
ThreadLocal 主要是用来实现线程间的隔离，
本质上是用私有的副本换取无锁的并发

每个线程内部维护了一个ThreadLocalMap用来存储线程的局部变量，作为独立的数据副本，避免共享冲突
key 是 ThreadLocal，value 是副本。
这样一来同一个线程就可以创建多个ThreadLocal变量，但每个线程访问的都是自己的数据副本，避免共享冲突

可以用来存储线程全局上下文的一些参数，如用户信息、traceId、事务对象，作为一个公共入口

ThreadLocal 为什么会造成内存泄漏
主要是由于ThreadLocal内部维护的map的数据结构和java 的 gc 机制导致

Map 的 key 是 threadlocal的弱引用，而value是强引用。
在java 的 gc 逻辑中，弱引用在没有被外部引用的情况下，gc中会被认为可以释放，

因此会导致map中key被释放为null，而value仍存在的问题，就可能造成内存泄漏

因此使用完ThreadLocal后最好手动调用remove方法，释放对应的value空间。
使用完 ThreadLocal 后，主动调用 remove() 清理当前线程的副本。

// 定义一个线程局部变量
private static ThreadLocal<Integer> local = ThreadLocal.withInitial(() -> 0);

public void doSomething() {
    try {
        local.set(10);   // 给当前线程设置副本
        // 使用逻辑
        int value = local.get(); // 获取当前线程的副本
    } finally {
        local.remove();  // 使用完后清理，避免内存泄漏
    }
}
