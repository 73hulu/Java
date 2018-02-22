# DualPivotQuicksort
<!-- toc -->
<!-- tocstop -->

类名翻译过来就是“双轴快速排序”，应用于`Arrays.sort`的系列方法中。类结构如下：

![DualPivotQuicksort](http://ovn0i3kdg.bkt.clouddn.com/DualPivotQuickSort.png?imageView/2/w/400)


算法是由Vladimir Yaroslavskiy在2009年研究出来的，并在2011年发布在了Java1.7。所有对外提供的都是`package-private`即`protected`，包内访问，由`Arrays.sort`方法调用。


该类只提供`Sort`，重载了不同的方法用来解决数据类型的问题。例如下面这个方法被`Arrays.sort`方法调用：

```Java
/**
* 给指定数组的指定范围排序
* @param a 指定的数组
* @param left 指定范围的第一个元素(包括)
* @param right 指定范围的最后一个元素(不包括)
*/
static void sort(int[] a, int left, int right,
                    int[] work, int workBase, int workLen) {
       // Use Quicksort on small arrays
       //当数组大小小于286的时候，直接使用快排
       if (right - left < QUICKSORT_THRESHOLD) {
           sort(a, left, right, true);
           return;
       }

       /**
       * run[i] 意味着第i个有序数列开始的位置，（升序或者降序）
       **/
       int[] run = new int[MAX_RUN_COUNT + 1];
       int count = 0; run[0] = left;

        // 检查数组是不是已经接近有序状态
       for (int k = left; k < right; run[count] = k) {
           if (a[k] < a[k + 1]) { // ascending
               while (++k <= right && a[k - 1] <= a[k]);
           } else if (a[k] > a[k + 1]) { // descending
               while (++k <= right && a[k - 1] >= a[k]);
               //如果是降序的，找出k之后，把数列倒置
               for (int lo = run[count] - 1, hi = k; ++lo < --hi; ) {
                   int t = a[lo]; a[lo] = a[hi]; a[hi] = t;
               }
           } else { // equal
               for (int m = MAX_RUN_LENGTH; ++k <= right && a[k - 1] == a[k]; ) {
                 // 数列中有至少MAX_RUN_LENGTH的数据相等的时候，直接使用快排。
                // 这里为什么这么处理呢？
                   if (--m == 0) {
                       sort(a, left, right, true);
                       return;
                   }
               }
           }

           /**
           * 数组并非高度有序，使用快速排序,因为数组中有序数列的个数超过了MAX_RUN_COUNT
           */
           if (++count == MAX_RUN_COUNT) {
               sort(a, left, right, true);
               return;
           }
       }

      //检查特殊情况
       // Implementation note: variable "right" is increased by 1.
       if (run[count] == right++) { // 最后一个有序数列只有最后一个元素
           run[++count] = right;// 那给最后一个元素的后面加一个哨兵
       } else if (count == 1) {  // 整个数组中只有一个有序数列，说明数组已经有序啦，不需要排序了
           return;
       }

       /**
        * 创建合并用的临时数组。
        * 注意： 这里变量right被加了1，它在数列最后一个元素位置+1的位置
        * 这里没看懂，没发现后面的奇数处理和偶数处理有什么不同
        */      
       byte odd = 0;
       for (int n = 1; (n <<= 1) < count; odd ^= 1);


       int[] b;                 // temp array; alternates with a
       int ao, bo;              // array offsets from 'left'
       int blen = right - left; // space needed for b
       if (work == null || workLen < blen || workBase + blen > work.length) {
           work = new int[blen];
           workBase = 0;
       }
       if (odd == 0) {
           System.arraycopy(a, left, work, workBase, blen);
           b = a;
           bo = 0;
           a = work;
           ao = workBase - left;
       } else {
           b = work;
           ao = 0;
           bo = workBase - left;
       }

       // 合并
      // 最外层循环，直到count为1，也就是栈中待合并的序列只有一个的时候，标志合并成功
      // a 做原始数组，b 做目标数组
       for (int last; count > 1; count = last) {
          // 遍历数组，合并相邻的两个升序序列
           for (int k = (last = 0) + 2; k <= count; k += 2) {
                // 合并run[k-2] 与 run[k-1]两个序列
               int hi = run[k], mi = run[k - 1];
               for (int i = run[k - 2], p = i, q = mi; i < hi; ++i) {
                 // 这里我给源码加了一个括号，这样好理解一点。 之前总觉得它会出现数组越界问题，
                // 后来加了这个括号之后发现是没有问题的
                   if (q >= hi || p < mi && a[p + ao] <= a[q + ao]) {
                       b[i + bo] = a[p++ + ao];
                   } else {
                       b[i + bo] = a[q++ + ao];
                   }
               }
               // 这里把合并之后的数列往前移动
               run[++last] = hi;
           }
            // 如果栈的长度为奇数，那么把最后落单的有序数列copy过对面
           if ((count & 1) != 0) {
               for (int i = right, lo = run[count - 1]; --i >= lo;
                   b[i + bo] = a[i + ao]
               );
               run[++last] = right;
           }
           //临时数组，与原始数组对调，保持a做原始数组，b 做目标数组
           int[] t = a; a = b; b = t;
           int o = ao; ao = bo; bo = o;
       }
   }

```
首先，当数组长度小于`QUICKSORT_THRESHOLD`，调用的是另一个对于int类型数组的排序方法，在这个方法里，优先使用的是快速排序而不是归并排序，具体的可以参照下面方法的具体介绍，这里先略过。


### private static void sort(int[] a, int left, int right, boolean leftmost){...}
参数`leftmost`表示指定的范围是否在数组的最左边。该方法的排序策略如下：
1. 对于很小的数组（长度小于27），会使用插入排序。
2. 选择两个点P1, P2作为轴心，比如我们可以使用第一个元素和最后一个元素。
3. P1必须比P2要小，否则将这两个元素交换，现在将整个数组分为四部分：
  1. 第一部分：比P1小的元素。
  2. 第二部分：比P1大但是比P2小的元素。
  3. 第三部分：比P2大的元素。
  4. 第四部分：尚未比较的部分。

在开始比较前，除了轴点，其余元素几乎都在第四部分，直到比较完之后第四部分没有元素。
4. 从第四部分选出一个元素a[K]，与两个轴心比较，然后放到第一二三部分中的一个。
5. 移动L，K，G指向。
6. 重复 4 5 步，直到第四部分没有元素。
7. 将P1与第一部分的最后一个元素交换。将P2与第三部分的第一个元素交换。
8. 递归的将第一二三部分排序。

图示：

![DualPivotQuickSort](http://ovn0i3kdg.bkt.clouddn.com/DualPivotQuickSort.sort.png)

算法实现如下：

```Java
/**
 * 使用双轴快速排序给指定数组的指定范围排序
 * @param a 参与排序的数组
 * @param left 范围内最左边的元素的位置(包括该元素)
 * @param right 范围内最右边的元素的位置(包括该元素)
 * @param leftmost 指定的范围是否在数组的最左边
 */
private static void sort(int[] a, int left, int right, boolean leftmost) {
    int length = right - left + 1;

    // 小数组使用插入排序
    if (length < INSERTION_SORT_THRESHOLD) {
        if (leftmost) {
            /**
             * 经典的插入排序算法，不带哨兵。做了优化，在leftmost情况下使用
             */
            for (int i = left, j = i; i < right; j = ++i) {
                int ai = a[i + 1];
                while (ai < a[j]) {
                    a[j + 1] = a[j];
                    if (j-- == left) {
                        break;
                    }
                }
                a[j + 1] = ai;
            }
        } else {
            /**
            * 首先跨过开头的升序的部分
            */
            do {
                if (left >= right) {
                    return;
                }
            } while (a[++left] >= a[left - 1]);

            /**
             * 这里用到了成对插入排序方法，它比简单的插入排序算法效率要高一些
             * 因为这个分支执行的条件是左边是有元素的
             * 所以可以直接从left开始往前查找。
             */
            for (int k = left; ++left <= right; k = ++left) {
                int a1 = a[k], a2 = a[left];

                //保证a1>=a2
                if (a1 < a2) {
                    a2 = a1; a1 = a[left];
                }
                //先把两个数字中较大的那个移动到合适的位置
                while (a1 < a[--k]) {
                    a[k + 2] = a[k];//这里每次需要向左移动两个元素
                }
                a[++k + 1] = a1;
                //再把两个数字中较小的那个移动到合适的位置
                while (a2 < a[--k]) {
                    a[k + 1] = a[k];//这里每次需要向左移动一个元素
                }
                a[k + 1] = a2;
            }
            int last = a[right];

            while (last < a[--right]) {
                a[right + 1] = a[right];
            }
            a[right + 1] = last;
        }
        return;
    }

    // length / 7 的一种低复杂度的实现, 近似值(length * 9 / 64 + 1)
    int seventh = (length >> 3) + (length >> 6) + 1;

    // 对5段靠近中间位置的数列排序，这些元素最终会被用来做轴(下面会讲)
    // 他们的选定是根据大量数据积累经验确定的
    int e3 = (left + right) >>> 1; // 中间值
    int e2 = e3 - seventh;
    int e1 = e2 - seventh;
    int e4 = e3 + seventh;
    int e5 = e4 + seventh;

    //这里是手写的冒泡排序，没有for循环
    if (a[e2] < a[e1]) { int t = a[e2]; a[e2] = a[e1]; a[e1] = t; }

    if (a[e3] < a[e2]) { int t = a[e3]; a[e3] = a[e2]; a[e2] = t;
        if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
    }
    if (a[e4] < a[e3]) { int t = a[e4]; a[e4] = a[e3]; a[e3] = t;
        if (t < a[e2]) { a[e3] = a[e2]; a[e2] = t;
            if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
        }
    }
    if (a[e5] < a[e4]) { int t = a[e5]; a[e5] = a[e4]; a[e4] = t;
        if (t < a[e3]) { a[e4] = a[e3]; a[e3] = t;
            if (t < a[e2]) { a[e3] = a[e2]; a[e2] = t;
                if (t < a[e1]) { a[e2] = a[e1]; a[e1] = t; }
            }
        }
    }

    //指针
    int less  = left;  // 中间区域的首个元素的位置
    int great = right; //右边区域的首个元素的位置

    if (a[e1] != a[e2] && a[e2] != a[e3] && a[e3] != a[e4] && a[e4] != a[e5]) {
        /*
         * 使用5个元素中的2，4两个位置，他们两个大致处在四分位的位置上。
         * 需要注意的是pivot1 <= pivot2
         */
        int pivot1 = a[e2];
        int pivot2 = a[e4];

        /*
         * The first and the last elements to be sorted are moved to the
         * locations formerly occupied by the pivots. When partitioning
         * is complete, the pivots are swapped back into their final
         * positions, and excluded from subsequent sorting.
         * 第一个和最后一个元素被放到两个轴所在的位置。当阶段性的分段结束后
         * 他们会被分配到最终的位置并从子排序阶段排除
         */
        a[e2] = a[left];
        a[e4] = a[right];

        /*
         * 跳过一些队首的小于pivot1的值，跳过队尾的大于pivot2的值
         */
        while (a[++less] < pivot1);
        while (a[--great] > pivot2);

        /*
         * Partitioning:
         *
         *   left part           center part                   right part
         * +--------------------------------------------------------------+
         * |  < pivot1  |  pivot1 <= && <= pivot2  |    ?    |  > pivot2  |
         * +--------------------------------------------------------------+
         *               ^                          ^       ^
         *               |                          |       |
         *              less                        k     great
         *
         * Invariants:
         *
         *              all in (left, less)   < pivot1
         *    pivot1 <= all in [less, k)     <= pivot2
         *              all in (great, right) > pivot2
         *
         * Pointer k is the first index of ?-part.
         */
        outer:
        for (int k = less - 1; ++k <= great; ) {
            int ak = a[k];
            if (ak < pivot1) { // Move a[k] to left part
                a[k] = a[less];
                /*
                 * 这里考虑的好细致，"a[i] = b; i++"的效率要好过
                 * 'a[i++] = b'
                 */
                a[less] = ak;
                ++less;
            } else if (ak > pivot2) { // k遇到great本次分割
                while (a[great] > pivot2) {
                    if (great-- == k) {
                        break outer;
                    }
                }
                if (a[great] < pivot1) { // a[great] <= pivot2
                    a[k] = a[less];
                    a[less] = a[great];
                    ++less;
                } else { // pivot1 <= a[great] <= pivot2
                    a[k] = a[great];
                }
                /*
                 * 同上，用"a[i]=b;i--"代替"a[i--] = b"
                 */
                a[great] = ak;
                --great;
            }
        }// 分割阶段结束出来的位置,上一个outer结束的位置

        // 把两个放在外面的轴放回他们应该在的位置上
        a[left]  = a[less  - 1]; a[less  - 1] = pivot1;
        a[right] = a[great + 1]; a[great + 1] = pivot2;

        // 把左边和右边递归排序，跟普通的快速排序差不多
        sort(a, left, less - 2, leftmost);
        sort(a, great + 2, right, false);

        /*
         * If center part is too large (comprises > 4/7 of the array),
         * swap internal pivot values to ends.
         * 如果中心区域太大，超过数组长度的 4/7。就先进行预处理，再参与递归排序。
         * 预处理的方法是把等于pivot1的元素统一放到左边，等于pivot2的元素统一
         * 放到右边,最终产生一个不包含pivot1和pivot2的数列，再拿去参与快排中的递归。
         */
        if (less < e1 && e5 < great) {
            /*
             * Skip elements, which are equal to pivot values.
             */
            while (a[less] == pivot1) {
                ++less;
            }

            while (a[great] == pivot2) {
                --great;
            }

            /*
             * Partitioning:
             *
             *   left part         center part                  right part
             * +----------------------------------------------------------+
             * | == pivot1 |  pivot1 < && < pivot2  |    ?    | == pivot2 |
             * +----------------------------------------------------------+
             *              ^                        ^       ^
             *              |                        |       |
             *             less                      k     great
             *
             * Invariants:
             *
             *              all in (*,  less) == pivot1
             *     pivot1 < all in [less,  k)  < pivot2
             *              all in (great, *) == pivot2
             *
             * Pointer k is the first index of ?-part.
             */
            outer:
            for (int k = less - 1; ++k <= great; ) {
                int ak = a[k];
                if (ak == pivot1) { // Move a[k] to left part
                    a[k] = a[less];
                    a[less] = ak;
                    ++less;
                } else if (ak == pivot2) { // Move a[k] to right part
                    while (a[great] == pivot2) {
                        if (great-- == k) {
                            break outer;
                        }
                    }
                    if (a[great] == pivot1) { // a[great] < pivot2
                        a[k] = a[less];
                        /*
                         * Even though a[great] equals to pivot1, the
                         * assignment a[less] = pivot1 may be incorrect,
                         * if a[great] and pivot1 are floating-point zeros
                         * of different signs. Therefore in float and
                         * double sorting methods we have to use more
                         * accurate assignment a[less] = a[great].
                         */
                        a[less] = pivot1;
                        ++less;
                    } else { // pivot1 < a[great] < pivot2
                        a[k] = a[great];
                    }
                    a[great] = ak;
                    --great;
                }
            }// outer结束的位置
        }

        // Sort center part recursively
        sort(a, less, great, false);

    } else { // 这里选取的5个元素刚好相等，使用传统的3-way快排
          /*
          * 在5个元素中取中值
          */
        int pivot = a[e3];

        /*
         * Partitioning degenerates to the traditional 3-way
         * (or "Dutch National Flag") schema:
         *
         *   left part    center part              right part
         * +-------------------------------------------------+
         * |  < pivot  |   == pivot   |     ?    |  > pivot  |
         * +-------------------------------------------------+
         *              ^              ^        ^
         *              |              |        |
         *             less            k      great
         *
         * Invariants:
         *
         *   all in (left, less)   < pivot
         *   all in [less, k)     == pivot
         *   all in (great, right) > pivot
         *
         * Pointer k is the first index of ?-part.
         */
        for (int k = less; k <= great; ++k) {
            if (a[k] == pivot) {
                continue;
            }
            int ak = a[k];
            if (ak < pivot) { // 把a[k]移动到左边去，把center区向右滚动一个单位
                a[k] = a[less];
                a[less] = ak;
                ++less;
            } else { // a[k] > pivot - 把a[k]移动到右边
                while (a[great] > pivot) {// 先找到右边最后一个比pivot小的值
                    --great;
                }
                if (a[great] < pivot) { // a[great] <= pivot ，把他移到左边
                    a[k] = a[less];
                    a[less] = a[great];
                    ++less;
                } else {  // a[great] == pivot //如果相等，中心区直接扩展
                  /*
                   * 这里因为是整型值，所以a[k] == a[less] == pivot;
                   */
                    a[k] = pivot;
                }
                a[great] = ak;
                --great;
            }
        }

        /*
         * 左右两边还没有完全排序，所以递归解决
         * 中心区只有一个值，不再需要排序
         */
        sort(a, left, less - 1, leftmost);
        sort(a, great + 1, right, false);
    }
}
```



参考
* [DualPivotQuickSort 双轴快速排序 源码 笔记](http://www.jianshu.com/p/6d26d525bb96)
* [JDK源码解析(1)——数据数组排序：Arrays.sort()](http://blog.csdn.net/octopusflying/article/details/52388012)
* [Java源码解析-DualPivotQuicksort](http://blog.csdn.net/xjyzxx/article/details/18465661)
* [](http://blog.csdn.net/holmofy/article/details/71168530)
* [快速排序算法原理及实现（单轴快速排序、三向切分快速排序、双轴快速排序）](http://www.cnblogs.com/nullzx/p/5880191.html)
