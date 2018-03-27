Top-K问题
===

`Top-K`问题在数据分析中非常普遍的一个问题（在面试中也经常被问到），比如:    

> 从1亿个数字中，找出其中最大的10000个数。    

在一大堆数中求其前`k`大或前`k`小的问题，简称`Top-K`问题。而目前解决`Top-K`问题最有效的算法即是`BFPRT`算法，其又称为中位数的中位数算法，
该算法由`Blum`、`Floyd`、`Pratt`、`Rivest`、`Tarjan`提出，最坏时间复杂度为`O(n)`。

这个问题总共有几种解决方式:   

- 最容易的方法就是将数据全部排序

    在首次接触`TOP-K`问题时，我们的第一反应就是可以先对所有数据进行一次排序，然后取其前`k`即可，但是这么做有两个问题:    
    - 快速排序的平均复杂度为O(nlogn)，但最坏时间复杂度为O(n^2)，不能始终保证较好的复杂度。
    - 我们只需要前`k`大的，而对其余不需要的数也进行了排序，浪费了大量排序时间。

    而且在32位的机器上，每个`float`类型占4个字节，1亿个浮点数就要占用`400MB`的存储空间，所以对于内存的要求很高，而且不能一次将全部数据
    读入内存进行排序。

- 局部淘汰法

    该方法与排序方法类似，用一个容器保存前`10000`个数，然后将剩余的所有数字逐一与容器内的最小数字相比，如果所有后续的元素都比容器内的`10000`个数还小，那么容器内这个`10000`个数就是最大`10000`个数。如果某一后续元素比容器内最小数字大，则删掉容器内最小元素，并将该元素插入容器，最后遍历完这`1`亿个数，得到的结果容器中保存的数即为最终结果了。此时的时间复杂度为`O（n+m^2）`，其中`m`为容器的大小，即`10000`。   

- 分治法

    将`1`亿个数据分成`100`份，每份`100`万个数据，找到每份数据中最大的`10000`个，最后在剩下的`100*10000`个数据里面找出最大的`10000`个。如果`100`万数据选择足够理想，那么可以过滤掉`1`亿数据里面`99%`的数据。`100`万个数据里面查找最大的`10000`个数据的方法如下：用快速排序的方法，将数据分为`2`堆，如果大的那堆个数`N`大于`10000`个，继续对大堆快速排序一次分成`2`堆，如果大的那堆个数`N`大于`10000`个，继续对大堆快速排序一次分成`2`堆，如果大堆个数`N`小于`10000`个，就在小的那堆里面快速排序一次，找第`10000-n`大的数字；递归以上过程，就可以找到第`10000`大的数。一共需要101次这样的比较。

- `Hash`法    

    如果这`1`亿个书里面有很多重复的数，先通过`Hash`法，把这`1`亿个数字去重复，这样如果重复率很高的话，会减少很大的内存用量，从而缩小运算空间，然后通过分治法或最小堆法查找最大的`10000`个数。    

- 最小堆   

    首先读入前`10000`个数来创建大小为`10000`的最小堆，建堆的时间复杂度为`O（mlogm）`（`m`为数组的大小即为`10000`），然后遍历后续的数字，并于堆顶（最小）数字进行比较。如果比最小的数小，则继续读取后续数字；如果比堆顶数字大，则替换堆顶元素并重新调整堆为最小堆。整个过程直至`1`亿个数全部遍历完为止。然后按照中序遍历的方式输出当前堆中的所有`10000`个数字。该算法的时间复杂度为`O（nmlogm）`，空间复杂度是10000（常数）。    


解决`Top K`问题有两种思路:   

- 最直观:小顶堆`(大顶堆 -> 最小100个数)`
- 较高效:`Quick Select`算法.   

堆排序也是一个比较好的选择，可以维护一个大小为`k`的堆，时间复杂度为`O(nlogk)`。     

那是否还存在更有效的方法呢？受到[快速排序](https://github.com/CharonChui/AndroidNote/blob/master/JavaKnowledge/%E5%85%AB%E7%A7%8D%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95.md)的启发，
通过修改快速排序中主元的选取方法可以降低快速排序在最坏情况下的时间复杂度（即`BFPRT`算法）.

并且我们的目的只是求出前`k`，故递归的规模变小，速度也随之提高。下面来简单回顾下快速排序的过程，以升序为例:       

- 选取主元（首元素，尾元素或一个随机元素）
- 以选取的主元为分界点，把小于主元的放在左边，大于主元的放在右边
- 分别对左边和右边进行递归，重复上述过程



堆
---


小顶堆`(min-heap)`有个重要的性质——每个结点的值均不大于其左右孩子结点的值，则堆顶元素即为整个堆的最小值。
`JDK`中`PriorityQueue`实现了数据结构堆，通过指定`comparator`字段来表示小顶堆或大顶堆，默认为`null`，表示自然序`(natural ordering)`。

小顶堆解决`Top-K`问题的思路:小顶堆维护当前扫描到的最大100个数，其后每一次的扫描到的元素，若大于堆顶，则入堆，然后删除堆顶；
依此往复，直至扫描完所有元素。`Java`实现第`K`大整数代码如下：

```java
public class TopK<E extends Comparable> {
    private PriorityQueue<E> queue;
    //堆的最大容量
    private int maxSize; 

    public TopK(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalStateException();
        }
        this.maxSize = maxSize;
        this.queue = new PriorityQueue<>(maxSize, new Comparator<E>() {
            @Override
            public int compare(E o1, E o2) {
                // 最大堆用o2 - o1，最小堆用o1 - o2
                return (o1.compareTo(o2));
            }
        });
    }

    public void add(E e) {
        if (queue.size() < maxSize) {
            queue.add(e);
        } else {
            E peek = queue.peek();
            if (e.compareTo(peek) > 0) {
                queue.poll();
                queue.add(e);
            }
        }
    }

    public List<E> sortedList() {
        List<E> list = new ArrayList<>(queue);
        Collections.sort(list);
        return list;
    }

    public static void main(String[] args) {
        int[] array = {4, 5, 1, 6, 2, 7, 3, 8};
        TopK pq = new TopK(4);
        for (int n : array) {
            pq.add(n);
        }
        System.out.println(pq.sortedList());
    }
}
```

下面使用`Java`来实现     

- 限定数据大小。
- 若堆满，则插入过程中与堆顶元素比较，并做相应操作。
- 每次删除堆顶元素后堆做一次调整，保证最小堆特性。

```java
public class Heap {
    private int[] data;

    public Heap(int[] data) {
        this.data = data;
        buildHeap();
    }

    public void buildHeap() {
        for (int i = data.length / 2 - 1; i >= 0; i--) {
            heapity(i);
        }
    }

    public void heapity(int i) {
        int left = getLeft(i);
        int right = getRight(i);
        int smallIndex = i;
        if (left < data.length && data[left] < data[i])
            smallIndex = left;
        if (right < data.length && data[right] < data[smallIndex])
            smallIndex = right;
        if (smallIndex == i)
            return;
        swap(i, smallIndex);
        heapity(smallIndex);
    }

    public int getLeft(int i) {
        return ((i + 1) << 1) - 1;
    }

    public int getRight(int i) {
        return (i + 1) << 1;
    }

    public void swap(int i, int j) {
        data[i] ^= data[j];
        data[j] ^= data[i];
        data[i] ^= data[j];
    }

    public int getMin() {
        return data[0];
    }

    public void setMin(int i) {
        data[0] = i;
        heapity(0);
    }
}

public class TopK {        
    private static int[] topK(int[] data,int k){  
        int topk[]=new int[k];  
        for (int i = 0; i < k; i++) {  
            topk[i]=data[i];  
        }  
        Heap heap=new Heap(topk);  
        for (int j = k; j < data.length; j++) {  
            int min=heap.getMin();  
            if(data[j]>min)  
                heap.setMin(data[j]);  
        }  
        return topk;  
    }  
    public static void main(String[] args) {  
          int[] data = {33,86,59,46,84,76,1236,963};    
          int[] topk=topK(data,3);  
          for (int i : topk) {  
            System.out.print(i+",");  
        }  
    }  
}  

```




`BFPRT`算法
---

`BFPRT`算法步骤如下:     

本算法的最坏时间复杂度为`O(n)`，值得注意的是通过`BFPTR`算法将数组按第`K`小（大）的元素划分为两部分，而
这高低两部分不一定是有序的，通常我们也不需要求出顺序，而只需要求出前`K`大的或者前`K`小的。

在`BFPTR`算法中，仅仅是改变了快速排序`Partion`中的`pivot`值的选取，在快速排序中，我们始终选择第一个元
素或者最后一个元素作为`pivot`，而在`BFPTR`算法中，每次选择五分中位数的中位数作为`pivot`，这样做的目的
就是使得划分比较合理，从而避免了最坏情况的发生。算法步骤如下:     

- 将输入数组的n个元素划分为`n/5`组，每组5个元素，且至多只有一个组由剩下的`n%5`个元素组成。
- 寻找`n/5`个组中每一个组的中位数，首先对每组的元素进行插入排序，然后从排序过的序列中选出中位数。
- 对于上面一步中找出的`n/5`个中位数，递归进行步骤`（1）`和`（2）`，直到只剩下一个数即为这`n/5`个元素的中位数，找到中位数后并找到对应的下标`p`。
- 进行`Partion`划分过程，`Partion`划分中的`pivot`元素下标为`p`。
- 进行高低区判断即可。

下面为代码实现，其所求为前`K`小的数:    

```java
public class BFPRT {
    /**
     * 返回前K小的数
     *
     * @param arr
     * @param k
     * @return
     */
    public static int[] getMinKNumsByBFPRT(int[] arr, int k) {
        if (k < 1 || k > arr.length) {
            return arr;
        }
        int minKth = getMinKthByBFPRT(arr, k);
        int[] res = new int[k];
        int index = 0;
        for (int i = 0; i != arr.length; i++) {
            if (arr[i] < minKth) {
                res[index++] = arr[i];
            }
        }
        for (; index != res.length; index++) {
            res[index] = minKth;
        }
        return res;
    }

    /**
     * 返回数组中第K小的数
     *
     * @param arr
     * @param K
     * @return
     */
    public static int getMinKthByBFPRT(int[] arr, int K) {
        int[] copyArr = copyArray(arr);
        return select(copyArr, 0, copyArr.length - 1, K - 1);
    }

    public static int[] copyArray(int[] arr) {
        int[] res = new int[arr.length];
        for (int i = 0; i != res.length; i++) {
            res[i] = arr[i];
        }
        return res;
    }

    /**
     * 在数组上给一个 end - begin 的范围,在这个范围上,返回位于第 i 位置上的数
     *
     * @param arr
     * @param begin
     * @param end
     * @param i
     * @return
     */
    public static int select(int[] arr, int begin, int end, int i) {
        if (begin == end) {
            return arr[begin];
        }
        int pivot = medianOfMedians(arr, begin, end);
        int[] pivotRange = partition(arr, begin, end, pivot);
        if (i >= pivotRange[0] && i <= pivotRange[1]) {
            return arr[i];
        } else if (i < pivotRange[0]) {
            return select(arr, begin, pivotRange[0] - 1, i);
        } else if (i > pivotRange[1]) {
            return select(arr, pivotRange[1] + 1, end, i);
        }

        return 0;
    }

    /**
     * 求一个范围内的划分值
     *
     * @param arr
     * @param begin
     * @param end
     * @return
     */
    public static int medianOfMedians(int[] arr, int begin, int end) {
        int num = end - begin + 1;
        int offset = num % 5 == 0 ? 0 : 1;
        int[] mArr = new int[num / 5 + offset]; //所有中位数组成的数组
        for (int i = 0; i < mArr.length; i++) {
            int beginI = begin + i * 5;
            int endI = beginI + 4;
            mArr[i] = getMedian(arr, beginI, Math.min(endI, end));
        }
        return select(mArr, 0, mArr.length - 1, mArr.length / 2);
    }

    /**
     * @param arr
     * @param begin
     * @param end
     * @param pivotValue 基准值
     * @return 返回等于区 最左边的位置 和 最右的位置
     */
    public static int[] partition(int[] arr, int begin, int end, int pivotValue) {
        int small = begin - 1;
        int big = end + 1;
        int i = begin;
        while (i < big) {
            if (arr[i] < pivotValue) {
                small++;
                swap(arr, small, i);
                i++;
            } else if (arr[i] > pivotValue) {
                big--;
                swap(arr, i, big);
            } else {
                i++;
            }
        }
        return new int[]{small + 1, big - 1};
    }

    /**
     * 获取上中位数
     * eg 9 10 11 12  取10
     * eg 1 2 3 4 5 取3
     *
     * @param arr
     * @param begin
     * @param end
     * @return
     */
    public static int getMedian(int[] arr, int begin, int end) {
        insertionSort(arr, begin, end);
        return arr[(end - begin) / 2 + begin];
    }

    /**
     * 插入排序
     *
     * @param arr
     * @param begin
     * @param end
     */
    public static void insertionSort(int[] arr, int begin, int end) {
        for (int i = begin + 1; i != end + 1; i++) {
            for (int j = i; j != begin; j--) {
                if (arr[j - 1] > arr[j]) {
                    swap(arr, j - 1, j);
                } else {
                    break;
                }
            }
        }
    }

    public static void swap(int[] arr, int index1, int index2) {
        int tmp = arr[index1];
        arr[index1] = arr[index2];
        arr[index2] = tmp;
    }

    public static void printArray(int[] arr) {
        System.out.print("前10小的数: ");
        for (int i = 0; i != arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] arr = {6, 9, 1, 3, 1, 2, 2, 5, 6, 1, 3, 5, 9, 7, 2, 5, 6, 1, 9};
        printArray(getMinKNumsByBFPRT(arr, 10));
        System.out.println("第10小的数: " + getMinKthByBFPRT(arr, 10));
    }
}
```


----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 