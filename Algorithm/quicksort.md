快速排序使用分治法（Divide and conquer）策略来把一个串行（list）分为两个子串行（sub-lists）。每个子串行又递归地使用同样的方法继续分成两个子串行，以此类推，并且对每个子串行都使用快速排序。

快速排序是一种分而治之思想在排序算法上的典型应用。本质上来看，快速排序应该算是在冒泡排序基础上的递归分治法。快速排序的最坏运行情况是 O(n²)，比如说顺序数列的快排。但它的平摊期望时间是 O(n logn)，且 O(n logn) 记号中隐含的常数因子很小，比复杂度稳定等于 O(n logn) 的归并排序要小很多。所以，对绝大多数顺序性较弱的随机数列而言，快速排序总是优于归并排序。

示例：给定一组随机排列的数字，按从小到大（或从大到小）的顺序重新排列。

```
int[] arr = {2,4,6,8,10,12,14,16};
ArrayList<Integer> list = new ArrayList<>();
for (int n: arr) {
    list.add(n);
}
Collections.shuffle(list);
System.out.println(quickSort(list));
```

Java代码实现：

```
public ArrayList‹Integer› quickSort(ArrayList<Integer> list){
    int pivot;
    ArrayList<Integer> less = new ArrayList<>();
    ArrayList<Integer> great = new ArrayList<>();
    ArrayList<Integer> rst = new ArrayList<>();
    if (list.size() < 2) {
        return list;
    } else {
        pivot = list.get(list.size() / 2);
        for (int n: list) {
            if (n < pivot) {
                less.add(n);
            } else {
                great.add(n);
            }
        }
    }

    less = quickSort(less);
    great = quickSort(great);

    less.add(pivot);
    rst.addAll(less);
    rst.addAll(great);
    return rst;
}
```