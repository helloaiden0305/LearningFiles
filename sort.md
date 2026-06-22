# Java 常见排序算法 — 最直白的写法

---

## 一、总览

| 排序 | 最好 | 平均 | 最坏 | 空间 | 稳定？ |
|------|------|------|------|------|--------|
| 冒泡 | O(n) | O(n²) | O(n²) | O(1) | ✓ |
| 选择 | O(n²) | O(n²) | O(n²) | O(1) | ✗ |
| 插入 | O(n) | O(n²) | O(n²) | O(1) | ✓ |
| 快速 | O(n log n) | O(n log n) | O(n²) | O(log n) | ✗ |
| 归并 | O(n log n) | O(n log n) | O(n log n) | O(n) | ✓ |
| 堆排 | O(n log n) | O(n log n) | O(n log n) | O(1) | ✗ |
| 计数 | O(n+k) | O(n+k) | O(n+k) | O(k) | ✓ |

> n = 元素数，k = 数据范围

---

## 二、冒泡排序 Bubble Sort

**一句话：相邻两个比，大的往后浮，像冒泡泡。**

```java
public static void bubbleSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {          // 每轮冒出一个最大值到末尾
        for (int j = 0; j < arr.length - 1 - i; j++) {  // 从前往后两两比较
            if (arr[j] > arr[j + 1]) swap(arr, j, j + 1);
        }
    }
}
```

**记忆：** `i` 控制几轮，`j` 控制每轮比几次。每轮最大的沉到末尾，像冒泡泡。

**图解：**
```
[3, 1, 2, 5, 4]
第1轮: [1, 2, 3, 4, 5]  ← 5 沉到最后
第2轮: [1, 2, 3, 4, 5]  ← 4 归位
第3轮: [1, 2, 3, 4, 5]  ← 3 归位
第4轮: [1, 2, 3, 4, 5]  ← 2 归位
```

---

## 三、选择排序 Selection Sort

**一句话：从剩下的里面找最小的，跟当前位置换。**

```java
public static void selectionSort(int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int min = i;                                      // 假设当前位置就是最小
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[j] < arr[min]) min = j;               // 找更小的
        }
        swap(arr, i, min);                                // 最小的放到 i 位置
    }
}
```

**记忆：** 先找完，再换一次。每轮只换 1 次。

**图解：**
```
[3, 1, 2, 5, 4]
第1轮: 剩下 [3,1,2,5,4] 找最小=1 → 和位置0换 → [1, 3, 2, 5, 4]
第2轮: 剩下 [3,2,5,4] 找最小=2 → 和位置1换 → [1, 2, 3, 5, 4]
第3轮: 剩下 [3,5,4] 找最小=3 → 不换 → [1, 2, 3, 5, 4]
第4轮: 剩下 [5,4] 找最小=4 → 和位置3换 → [1, 2, 3, 4, 5]
```

---

## 四、插入排序 Insertion Sort

**一句话：像抓牌一样，把每张牌插到手里排好序的位置。**

```java
public static void insertionSort(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        int temp = arr[i];                                // 手里这张牌
        int j = i - 1;
        while (j >= 0 && arr[j] > temp) {                // 比手里大就往右挪
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = temp;                                // 找到位置，插进去
    }
}
```

**记忆：** 边找边挪。已有序时最快（几乎不用挪）。

**图解：**
```
[3, 1, 2, 5, 4]
第1轮: 抓 1，插在 3 前面 → [1, 3, 2, 5, 4]
第2轮: 抓 2，插在 3 前面 → [1, 2, 3, 5, 4]
第3轮: 抓 5，前面都比它小 → 不动 → [1, 2, 3, 5, 4]
第4轮: 抓 4，插在 5 前面 → [1, 2, 3, 4, 5]
```

---

## 五、快速排序 Quick Sort（双指针法）

**一句话：选基准，左找大的，右找小的，撞了就把基准放中间。**

```java
public static void quickSort(int[] arr) {
    quickSort(arr, 0, arr.length - 1);
}

private static void quickSort(int[] arr, int low, int high) {
    if (low >= high) return;
    int mid = partition(arr, low, high);                // 分好区
    quickSort(arr, low, mid - 1);                        // 排左边
    quickSort(arr, mid + 1, high);                       // 排右边
}

private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];                              // 选最后一个当基准
    int left = low, right = high - 1;

    while (left <= right) {
        while (left <= right && arr[left] <= pivot) left++;   // 左找大的
        while (left <= right && arr[right] > pivot) right--;  // 右找小的
        if (left < right) swap(arr, left, right);
    }
    swap(arr, left, high);                                // 基准放到中间
    return left;
}
```

**记忆：** 左找大的、右找小的、撞了停、基准放中间。

**图解：**
```
[3, 1, 2, 5, 4]  pivot=4
  左→ 找 >4 的，找到 5
  右← 找 <4 的，找到 2
  左 5 和 右 2 换 → [3, 1, 2, 5, 4]（3<4，左继续走）
  左→ 5，右← 1，撞了
  pivot 4 放中间 → [3, 1, 2, 4, 5]  （4 归位）

左边 [3, 1, 2] pivot=2
  左→ 找 >2 的，找到 3
  右← 找 <2 的，找到 1
  3 和 1 换 → [1, 3, 2, 4, 5]
  撞了，pivot 2 放中间 → [1, 2, 3, 4, 5]  （2 归位）

右边 [3] 一个元素 → 不用排
结果: [1, 2, 3, 4, 5] ✓
```

---

## 六、归并排序 Merge Sort

**一句话：分两半各自排好，再合并。**

```java
public static void mergeSort(int[] arr) {
    mergeSort(arr, new int[arr.length], 0, arr.length - 1);
}

private static void mergeSort(int[] arr, int[] temp, int left, int right) {
    if (left >= right) return;
    int mid = (left + right) / 2;
    mergeSort(arr, temp, left, mid);                       // 排左半
    mergeSort(arr, temp, mid + 1, right);                  // 排右半
    merge(arr, temp, left, mid, right);                    // 合并
}

private static void merge(int[] arr, int[] temp, int left, int mid, int right) {
    for (int k = left; k <= right; k++) temp[k] = arr[k]; // 先复制
    int i = left, j = mid + 1, k = left;
    while (i <= mid && j <= right)
        arr[k++] = temp[i] <= temp[j] ? temp[i++] : temp[j++];
    while (i <= mid) arr[k++] = temp[i++];               // 左半还有剩
}
```

**记忆：** 先分到单个元素，再两两合并。每轮谁小谁先放。

**图解：**
```
分: [3, 1, 2, 5, 4]
   → [3, 1, 2] | [5, 4]
   → [3, 1] | [2] | [5] | [4]
   → [3] | [1] | [2] | [5] | [4]

合: [1, 3] | [2] | [4, 5]
   → [1, 2, 3] | [4, 5]
   → [1, 2, 3, 4, 5] ✓
```

---

## 七、堆排序 Heap Sort

**一句话：建大顶堆，根就是最大的，换到末尾再调整。**

```java
public static void heapSort(int[] arr) {
    int n = arr.length;
    for (int i = n / 2 - 1; i >= 0; i--) heapify(arr, n, i);  // 建大顶堆
    for (int i = n - 1; i > 0; i--) {
        swap(arr, 0, i);                                      // 最大（根）换到末尾
        heapify(arr, i, 0);                                   // 调整剩下 i 个
    }
}

private static void heapify(int[] arr, int n, int i) {
    int largest = i, left = 2 * i + 1, right = 2 * i + 2;
    if (left < n && arr[left] > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;
    if (largest != i) {
        swap(arr, i, largest);
        heapify(arr, n, largest);                             // 递归调整
    }
}
```

**记忆：** 父节点永远比子节点大，每次把最大的换到末尾。

**图解：**
```
原数组 [3, 1, 2, 5, 4] 看成二叉树:
        5
      /   \
    3       4
   / \
  1   2

第1轮: 5 换到末尾 → [4, 1, 2, 5, 4]（5 归位），调整前 4 个
        4
      /   \
    3       1
   /
  2

第2轮: 4 换到末尾 → [2, 1, 4, 5, 4]（4 归位），调整前 3 个
        3
      /   \
    2       1

第3轮: 3 换到末尾 → [1, 2, 3, 4, 5]（3 归位）
        2
      /
    1

第4轮: 2 换到末尾 → [1, 2, 3, 4, 5]（2 归位）

结果: [1, 2, 3, 4, 5] ✓
```

---

## 八、计数排序 Counting Sort

**一句话：统计每个数出现几次，再按顺序展开。**

```java
public static int[] countingSort(int[] arr) {
    if (arr.length < 2) return arr;
    int max = Integer.MIN_VALUE, min = Integer.MAX_VALUE;
    for (int x : arr) { max = Math.max(max, x); min = Math.min(min, x); }
    int range = max - min + 1;
    int[] count = new int[range], out = new int[arr.length];
    for (int x : arr) count[x - min]++;                        // 统计频次
    for (int i = 1; i < range; i++) count[i] += count[i - 1]; // 累加
    for (int i = arr.length - 1; i >= 0; i--) {               // 反向放（稳定）
        out[count[arr[i] - min] - 1] = arr[i];
        count[arr[i] - min]--;
    }
    return out;
}
```

**记忆：** 不用比较，直接统计。整数且范围不大时最快。

**图解：**
```
[3, 1, 2, 1, 4]
count:  [0, 2, 1, 1, 1]  （1出现2次，2出现1次，...）
累加:   [0, 2, 3, 4, 5]
展开:   [1, 1, 2, 3, 4] ✓
```

---

## 九、工具方法

```java
static void swap(int[] a, int i, int j) {
    if (i != j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
}

static void print(int[] a) { System.out.println(java.util.Arrays.toString(a)); }

static int[] rand(int n, int min, int max) {
    int[] a = new int[n];
    for (int i = 0; i < n; i++) a[i] = (int)(Math.random() * (max - min + 1)) + min;
    return a;
}
```

---

## 十、测试

```java
public static void main(String[] args) {
    int[] a = rand(10, 1, 20);
    print(a);

    int[] t = a.clone(); bubbleSort(t);     print("冒泡: " + java.util.Arrays.toString(t));
    t = a.clone(); selectionSort(t);         print("选择: " + java.util.Arrays.toString(t));
    t = a.clone(); insertionSort(t);         print("插入: " + java.util.Arrays.toString(t));
    t = a.clone(); quickSort(t);             print("快排: " + java.util.Arrays.toString(t));
    t = a.clone(); mergeSort(t);             print("归并: " + java.util.Arrays.toString(t));
    t = a.clone(); heapSort(t);              print("堆排: " + java.util.Arrays.toString(t));
    t = countingSort(a);                     print("计数: " + java.util.Arrays.toString(t));
}
```

---

## 十一、面试速记

**怎么选？**

| 场景 | 用哪个 |
|------|--------|
| 数据接近有序 | 插入排序 O(n) |
| 通用场景 | 快速排序（常数小，最快） |
| 要求稳定 | 归并排序 |
| 空间有限 | 堆排序 O(1) 空间 |
| 整数小范围 | 计数排序 O(n) |

**Java 标准库：**
- `Arrays.sort(int[])` → 双轴快排
- `Arrays.sort(Object[])` → TimSort（归并+插入，稳定）

**为什么快排最快？** O(n log n) 平均 + 常数因子小 + 缓存友好 + 原地排序。

**归并 vs 快排？**
- 归并：稳定，O(n) 额外空间，适合链表
- 快排：不稳定，O(log n) 栈空间，适合数组
