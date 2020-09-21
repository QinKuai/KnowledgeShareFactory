## 侃侃算法EP7·说说排序

### 1. 前言

- 这个板块旨在记录一些日常中或是面试中会问到的算法和数据结构相关的内容，更多是给自己总结和需要的人分享。在内容部分可能由于我的阅历和实战经历不足，会有忽视或是写错的点，还望轻喷。



### 2. 内容

#### 2.1 排序的概念

  - 稳定性：指原来序列中相等的两个元素，经过排序后的顺序不变，如果发生了变化则称为不稳定。
  - 内排序与外排序：
        - 内排序：在排序的整个过程，排序的所有记录全部放在内存中。
        - 外排序：由于数据过多，不能同时全放在内存中，整个排序过程需要在内外存之间多次交换数据。



#### 2.2 冒泡排序

- 冒泡排序作为一种交换排序，基本想法就是通过两两比较相邻元素，如果反序则交换，直到没有反序的情况。
- **冒泡排序平均时间复杂度为O(N^2)**。

```java
// 最基本的思路
// 通过比较所有可能的一对元素，并通过交换实现排序
// 每一次外层循环实现一个元素的排序
public void bubbleSort(int[] nums){
    int tmp;
	for(int i = 0; i < nums.length; i++){
        for(int j = i + 1; j < nums.length; j++){
            if(nums[i] > nums[j]){
            	tmp = nums[j];
                nums[j] = nums[i];
                nums[i] = tmp;
            }
        }
    }
}

// 正式的冒泡
// 从底部将小值往上送
// 同样是每一层外层循环实现对一个元素的排序
public void bubbleSort(int[] nums){
    int tmp;
    for(int i = 0; i < nums.length; i++){
        for(int j = nums.length - 1; j > i; j--){
            if(nums[j - 1] > nums[j]){
                tmp = nums[j];
                nums[j] = nums[j - 1];
                nums[j - 1] = tmp;
            }
        }
    }
}

// 简单优化
// 主要是实现当序列已经排好后，快速结束
public void bubbleSort(int[] nums){
    int tmp;
    boolean flag = true;
    for(int i = 0; i < nums.length && flag; i++){
        for(int j = nums.length - 1; j > i; j--){
            flag = false;
            if(nums[j - 1] > nums[j]){
                tmp = nums[j];
                nums[j] = nums[j - 1];
                nums[j - 1] = tmp;
                
                flag = true;
            }
        }
    }
}
```



#### 2.3 选择排序

- 选择排序，思想是通过n-i次元素的比较，从n-i+1个元素中找出最小的，再与第i个元素交换（1<= i <=n）
- **平均时间复杂度为O(N^2)，但该算法优于冒泡排序，因为减少了交换的次数**。

```java
public void selectSort(int[] nums){
    int minIndex, tmp;
    for(int i = 0; i < nums.length; i++){
        minIndex = i;
        for(int j = i + 1; j < nums.length; j++){
            if(nums[minIndex] > nums[j]){
                minIndex = j;
            }
        }
        if(i != minIndex){
            // 交换元素
            tmp = nums[i];
            nums[i] = nums[minIndex];
            nums[minIndex] = tmp;
        }
    }
}
```



#### 2.4 直接插入排序

- 直接插入排序，思想是将一个元素插入到已经排序好的有序表中，等到新的有序表，类似于扑克牌排好后又摸了一张的逻辑。
- **平均时间复杂度为O(N^2)，综合性能优于上述两种排序算法**。

```java
public void insertSort(int[] nums){
    int tmp, j;
    for(int i = 1; i < nums.length; i++){
        if(nums[i] < nums[i - 1]){
            tmp = nums[i];
            // 除了限制值大小的比较，还需要限制j不能为负值
            for(j = i - 1; j >= 0 && nums[j] > tmp; j--){
                nums[j + 1] = nums[j];
            }
            nums[j + 1] = tmp;
        }
    }
}
```



#### 2.5 希尔排序

- 希尔排序，思想就是通过一个步长increment实现跳跃式排序，从基本有序到完全有序。
- 该排序有点类似分步法下的插入排序，increment在其中起着很重要的作用。
- **时间复杂度为O(N^(3/2))，排序不稳定**。

```java
public void shellSort(int[] nums){
    int tmp, increment = nums.length;
    while(increment > 1){
        // 关于步长如何确定，有很多方式，这里只是提供了一种
        increment = increment / 3 + 1;
        
        // 外层循环核心做的事就是遍历数组
        // 使得数组一次次从基本有序到完全有序
        int j;
        for(int i = increment + 1; i < nums.length; i++){
        	if(nums[i] < nums[i - increment]){
                // 插入排序流程逻辑
                tmp  = nums[i];
                for(j = i - increment; j >= 0 && tmp < nums[j]; j -= increment){
                    nums[j + increment] = nums[j];
                }
                nums[j + increment] = tmp;
            }    
        }
    }
}
```



#### 2.6 堆排序

- 堆排序，是对选择排序的一种改进，引入了一种数据结构概念，堆。
- 堆是一种独特的**完全二叉树**，
  - 其每个节点的值都**大于或者等于其左右子节点的值**，称为**大顶堆**。
  - 其每个节点的值都**小于或者等于其左右子节点的值**，称为**小顶堆**。
- 因此堆排序就是，**通过将待排序序列构造成大顶堆（或者小顶堆，这里以大顶堆为例），整个序列的最大值就是堆顶的根节点，移走后（从该数组的最后开始覆盖）剩下的节点重新构造成一个顶堆，以此类推就能等到有序序列了。**
- **时间复杂度为O(N*logN)，排序不稳定**，由于需要大量的比较，不适合待排序序列个数较少的情况。

```java
public void heapSort(int[] nums){
    // 为什么只选取nums.length / 2个节点？
    // 因为对于一棵堆（完全二叉树）的构建，实际上只需要节点个数除上2的向下取整的值
    // 其他节点都是这个堆的叶子节点，这是完全二叉树的特性
    for(int i = nums.length / 2 - 1; i >= 0; i--){
        heapAdjust(nums, i, nums.length - 1);
    }
    
    // 排序
    int tmp;
    for(int i = nums.length - 1; i > 0; i--){
        // swap堆顶和堆尾数据
        tmp = nums[i];
        nums[i] = nums[0];
        nums[0] = tmp;
        
        heapAdjust(nums, 0, i - 1);
    }
}

// 已知nums[s...m]除了nums[s]都符合堆的定义
// 该方法就是要将nums[s]纳入堆中
private void heapAdjust(int[] nums, int s, int m){
    int tmp = nums[s];
    // 这里使用循环是解决子节点还有自己的子节点的情况
    // 这种情况交换数据可能打破子节点的堆规则，因此还需再继续判断
    for(int i = (s << 1) + 1; i <= m; i <<= 1){
        // 确定左右子节点中谁最大，并得到索引
        if(i < m && nums[i] < nums[i + 1]){
            i++;
        }
        // 如果当前这个顶节点数值大于左右节点的最大值直接返回
        // 因为满足堆的定义
        if(tmp >= nums[i]){
            break;
        }
        //
        nums[s] = nums[i];
        s = i;
    }
    nums[s] = tmp;
}
```



#### 2.7 归并排序

- 归并排序，将元素数量为n的数组序列看作n个有序的子序列，然后两两合并，如此重复就能得到长度为n的有序序列。
- **时间复杂度为O(N*logN)，排序稳定**。
- 使用递归完成归并排序

```java
public void mergeSort(int[] nums){
    mergeSort0(nums, nums, 0, nums.length - 1);
}

private void mergeSort0(int[] nums, int[] target, int s, int t){
    int[] tmpArray = new int[nums.length];
    if(s == t){
        target[s] = nums[s];
    }else{
        int m = (s + t) >> 1;
        mergeSort0(nums, tmpArray, s, m);
        mergeSort0(nums, tmpArray, m + 1, t);
        merge0(tmpArray, target, s, m ,t);
    }
}

private void merge0(int[] nums, int[] target, int s, int m, int t){
    int i,j,k;
    for(j = m + 1, k = s; j <= t && s <= m; k++){
        if(nums[s] < nums[j]){
            target[k] = nums[s++];
        }else{
            target[k] = nums[j++];
        }
    }
    
    // 当s到m的部分有剩下的元素
    if(s <= m){
        for(int p = s; p <= m; p++){
            target[k++] = nums[p];
        }
    }
    
    // 当m+1到t的部分有剩下的元素
    if(j <= t){
        for(int p = j; j <= t; j++){
            target[k++] = nums[j];
        }
    }
}
```

- 使用非递归实现归并排序

```java
public void mergeSort(int[] nums){
    int[] tmpArray = new int[nums.length];
    int step = 1;
    while(step < nums.length){
        mergePass(nums, tmpArray, step, nums.length - 1);
        step <<= 1;
        mergePass(tmpArray, nums, step, nums.length - 1);
        step <<= 1;
    }
}

private void mergePass(int[] nums, int[] target, int step, int end){
    int i = 0, j;
    while(i <= end - 2 * step + 1){
        merge0(nums, target, i, i + step - 1, i + 2 * step - 1);
        i += 2 * step;
    }
    
    if(i < end - step){
        merge0(nums, target, i, i + step - 1, end);
    }else{
        for(int j = i; j <= end; j++){
            target[j] = nums[j];
        }
    }
}
```



#### 2.8 快速排序

- 快速排序，思想是通过一趟排序将待排记录分割成独立的两部分，其中一部分元素均比另一部分要小，再对这两部分执行同样的操作，最终即可得到有序序列。
- **时间复杂度是O(N*logN)，几乎可以说是最优的排序算法，但快排并不稳定**。

```java
public void quickSort(int[] nums){
    quickSort0(nums, 0, nums.length - 1);
}

private void quickSort0(int[] nums, int start, int end){
    int middle;
    if(start < end){
        middle = partition(nums, start, end);
        
        quickSort0(nums, start, middle - 1);
        quickSort0(nums, middle + 1, end);
    }
}

private int partition(int[] nums, int start, int end){
    int middle = nums[start];
    while(start < end){
        while(start < end && nums[end] >= middle){
            end--;
        }
        swap(nums, start, end);
        while(start < end && nums[start] <= middle){
            start++;
        }
        swap(nums, start, end);
    }
    return start;
}

private void swap(int nums, int index1, int index2){
    int tmp = nums[index1];
    nums[index1] = nums[index2];
    nums[index2] = tmp;
}
```

