选择排序选择排序是一种简单直观的排序算法，适用于无序数组的排序，但是由于这种简单查找必须来回检查列表中的每个元素，这意味着它的时间复杂度为$O(n^2)$，因此当问题规模增大以后，其速度不快的劣势就会显现出来。

示例：给定一列数组元素，按从小到大（或从大到小）方式排列。

```
ArrayList‹Integer› list = new ArrayList<>();
Random r = new Random();
StringBuilder sb = new StringBuilder(100);
for (int i = 0; i < 30; i++) {
    int rst = r.nextInt(100);
    list.add(rst);
} //create a random list
for (int n: list) {
    sb.append(n).append(" ");
}
System.out.println("Now the random list is: \n" + sb.toString());
System.out.println("List after selection sorting: \n" + selctionSort(list));
```

Java代码实现：

```
public String selctionSort(ArrayList<Integer> list) {
    StringBuilder sb = new StringBuilder(1024);
    int smallest;
    while (list.size() > 0) {
        smallest = findSmallest(list);
        sb.append(list.get(smallest)).append(" ");
        list.remove(smallest); //renew the list until the size becomes 0
    }
    return sb.toString();
}

private int findSmallest(ArrayList<Integer> list){
    int smallest = list.get(0);
    int index = 0;
    for (int n: list) {
        if (smallest >= n) {
            smallest = n;
            index = list.indexOf(smallest);
        }
    }
    return index;
}
```