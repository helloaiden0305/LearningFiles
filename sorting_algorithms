# Java 常见排序算法实现 + 复杂度对比

---

## 一、复杂度总览

| 排序算法 | 最好时间 | 平均时间 | 最坏时间 | 空间复杂度 | 稳定性 |
|---------|---------|---------|---------|-----------|--------|
| 冒泡排序 | O(n) | O(n²) | O(n²) | O(1) | 稳定 |
| 选择排序 | O(n²) | O(n²) | O(n²) | O(1) | 不稳定 |
| 插入排序 | O(n) | O(n²) | O(n²) | O(1) | 稳定 |
| 快速排序 | O(n log n) | O(n log n) | O(n²) | O(log n) | 不稳定 |
| 归并排序 | O(n log n) | O(n log n) | O(n log n) | O(n) | 稳定 |
| 堆排序 | O(n log n) | O(n log n) | O(n log n) | O(1) | 不稳定 |
| 计数排序 | O(n+k) | O(n+k) | O(n+k) | O(k) | 稳定 |
| 桶排序 | O(n+k) | O(n+k) | O(n²) | O(n+k) | 稳定 |
| 希尔排序 | O(n log n) | O(n^1.3) | O(n²) | O(1) | 不稳定 |

> **n** = 元素个数，**k** = 数据范围

---

## 二、冒泡排序 Bubble Sort

**思路：** 相邻元素两两比较，每一轮把最大的"冒"到末尾。

```java
public static void bubbleSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;  // 提前终止优化
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr, j, j + 1);
                swapped = true;
            }
        }
        if (!swapped) break;  // 已有序，直接退出
    }
}
```

---

## 三、选择排序 Selection Sort

**思路：** 每次从未排序部分选出最小的元素，放到已排序末尾。

```java
public static void selectionSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        swap(arr, i, minIndex);
    }
}
```

---

## 四、插入排序 Insertion Sort

**思路：** 把每个元素插入到已排序部分的正确位置。数据接近有序时非常高效。

```java
public static void insertionSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    int n = arr.length;
    for (int i = 1; i < n; i++) {
        int temp = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > temp) {
            arr[j + 1] = arr[j];  // 后移
            j--;
        }
        arr[j + 1] = temp;  // 插入
    }
}
```

---

## 五、快速排序 Quick Sort

**思路：** 选一个 pivot，把小于 pivot 的放左边，大于的放右边，然后递归排序左右两部分。

```java
public static void quickSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    quickSort(arr, 0, arr.length - 1);
}

private static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pivotIndex = partition(arr, low, high);
        quickSort(arr, low, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, high);
    }
}

private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];  // 选最后一个元素作为 pivot
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    swap(arr, i + 1, high);
    return i + 1;
}
```

---

## 六、归并排序 Merge Sort

**思路：** 分治法 — 把数组不断二分，分别排序后合并。

```java
public static void mergeSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    int[] temp = new int[arr.length];
    mergeSort(arr, temp, 0, arr.length - 1);
}

private static void mergeSort(int[] arr, int[] temp, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, temp, left, mid);
        mergeSort(arr, temp, mid + 1, right);
        merge(arr, temp, left, mid, right);
    }
}

private static void merge(int[] arr, int[] temp, int left, int mid, int right) {
    // 复制到临时数组
    System.arraycopy(arr, left, temp, left, right - left + 1);
    
    int i = left;      // 左半部分指针
    int j = mid + 1;   // 右半部分指针
    int k = left;      // 合并指针
    
    while (i <= mid && j <= right) {
        if (temp[i] <= temp[j]) {
            arr[k++] = temp[i++];
        } else {
            arr[k++] = temp[j++];
        }
    }
    while (i <= mid) arr[k++] = temp[i++];
    // 右半部分剩余的不需要处理，已在原位置
}
```

---

## 七、堆排序 Heap Sort

**思路：** 先建大顶堆，每次把堆顶（最大值）和末尾交换，再调整堆。

```java
public static void heapSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    int n = arr.length;
    
    // 1. 建大顶堆
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }
    
    // 2. 逐个提取最大值
    for (int i = n - 1; i > 0; i--) {
        swap(arr, 0, i);       // 堆顶放到末尾
        heapify(arr, i, 0);    // 重新调整堆
    }
}

private static void heapify(int[] arr, int n, int i) {
    int largest = i;          // 根节点
    int left = 2 * i + 1;     // 左子节点
    int right = 2 * i + 2;    // 右子节点
    
    if (left < n && arr[left] > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;
    
    if (largest != i) {
        swap(arr, i, largest);
        heapify(arr, n, largest);  // 递归调整
    }
}
```

---

## 八、希尔排序 Shell Sort

**思路：** 插入排序的改进版，先用间隔（gap）分组插入排序，逐步缩小 gap 到 1。

```java
public static void shellSort(int[] arr) {
    if (arr == null || arr.length < 2) return;
    int n = arr.length;
    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; i++) {
            int temp = arr[i];
            int j = i;
            while (j >= gap && arr[j - gap] > temp) {
                arr[j] = arr[j - gap];
                j -= gap;
            }
            arr[j] = temp;
        }
    }
}
```

---

## 九、计数排序 Counting Sort

**思路：** 统计每个值出现的次数，再按顺序展开。适用于整数且数据范围不大的场景。

```java
public static int[] countingSort(int[] arr) {
    if (arr == null || arr.length < 2) return arr;
    
    int max = Integer.MIN_VALUE, min = Integer.MAX_VALUE;
    for (int num : arr) {
        max = Math.max(max, num);
        min = Math.min(min, num);
    }
    
    int range = max - min + 1;
    int[] count = new int[range];
    int[] output = new int[arr.length];
    
    // 统计频次
    for (int num : arr) count[num - min]++;
    
    // 前缀和
    for (int i = 1; i < range; i++) count[i] += count[i - 1];
    
    // 反向填充（保证稳定性）
    for (int i = arr.length - 1; i >= 0; i--) {
        output[count[arr[i] - min] - 1] = arr[i];
        count[arr[i] - min]--;
    }
    
    return output;
}
```

---

## 十、辅助方法

```java
private static void swap(int[] arr, int i, int j) {
    if (i != j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}

// 工具：打印数组
public static void printArr(int[] arr) {
    StringBuilder sb = new StringBuilder("[");
    for (int i = 0; i < arr.length; i++) {
        sb.append(arr[i]).append(i == arr.length - 1 ? "" : ", ");
    }
    System.out.println(sb + "]");
}

// 工具：生成随机数组
public static int[] randomArr(int n, int min, int max) {
    int[] arr = new int[n];
    for (int i = 0; i < n; i++) {
        arr[i] = (int) (Math.random() * (max - min + 1)) + min;
    }
    return arr;
}
```

---

## 十一、测试代码

```java
public static void main(String[] args) {
    int[] arr = randomArr(10, 1, 100);
    System.out.print("原始: ");
    printArr(arr);
    
    int[] a1 = arr.clone(); bubbleSort(a1);
    System.out.print("冒泡: "); printArr(a1);
    
    int[] a2 = arr.clone(); selectionSort(a2);
    System.out.print("选择: "); printArr(a2);
    
    int[] a3 = arr.clone(); insertionSort(a3);
    System.out.print("插入: "); printArr(a3);
    
    int[] a4 = arr.clone(); quickSort(a4);
    System.out.print("快速: "); printArr(a4);
    
    int[] a5 = arr.clone(); mergeSort(a5);
    System.out.print("归并: "); printArr(a5);
    
    int[] a6 = arr.clone(); heapSort(a6);
    System.out.print("堆排: "); printArr(a6);
    
    int[] a7 = arr.clone(); shellSort(a7);
    System.out.print("希尔: "); printArr(a7);
    
    int[] a8 = countingSort(arr);
    System.out.print("计数: "); printArr(a8);
}
```

---

## 十二、面试常问

**Q1：什么时候用插入排序？**
> 数据量小、基本有序时。时间复杂度退化为 O(n)，是最快的 O(n²) 算法。Java 的 `Arrays.sort()` 对小数组也会切换到插入排序。

**Q2：为什么快速排序是最常用的？**
> 虽然最坏 O(n²)，但平均 O(n log n)，且常数因子小、缓存友好、原地排序。实际性能通常优于归并和堆排序。

**Q3：归并 vs 快排的区别？**
> - 归并：稳定，但需要 O(n) 额外空间；适合链表排序
> - 快排：不稳定，原地排序；适合数组排序

**Q4：计数排序的适用范围？**
> 只适用于**整数**且**数据范围 k 不大**的场景。比如对学生分数排序、年龄排序等。如果范围很大（如 long），空间开销会爆炸。

**Q5：Java 标准库用的什么排序？**
> - `Arrays.sort(int[])` → 双轴快排（Dual-Pivot Quicksort）
> - `Arrays.sort(Object[])` → TimSort（归并 + 插入的混合，稳定）
> - `Collections.sort()` → 底层也是 TimSort
