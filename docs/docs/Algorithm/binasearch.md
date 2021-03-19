二分查找是一种速度比较快的搜索算法，其输入是一个**有序**的元素列表。如果要查找的元素包含在列表中，二分查找返回其位置；否则返回null。二分查找的特点在于，每次都能排除一半的元素，从而快速缩小查找范围。因此，二分查找的时间复杂度为$O(log_2\ n)$

示例：给定一个有序数组，从数组中找到指定元素，找不到则提示元素不存在。

```
int num = new Random().nextInt(30);
int[] list = {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14};
if (binary_search(list,num) == -1) {
    System.out.println(r_num + " is not here!");
} else {
    System.out.println(r_num + " exists! Position: " + result);
}
```
            
Java代码实现：

```
public int binarySearch(int[] list, int item) {
    int low = 0;
    int high = list.length - 1;
    int mid;
    int guess;
    while (low < high) {
        mid = (low + high) / 2;
        guess = list[mid]; //set the middle number for comparison
        if (guess == item) {
            return guess;
        } else if (guess > item) {
            high = mid - 1; //if the middle is greater than item, then adjust the high bound to mid - 1
        } else if (guess < item) {
            low = mid + 1;  //if the middle is less than item, then adjust the low bound to mid + 1
        }
    }
    return -1;
}
```