### 二分查找

```java
import java.util.Arrays;
public class BinarySearch
{
    public static int rank(int key,int[] a){
        //数组必须是有序的
        int lo = 0;
        int hi = a.length - 1;
        while(lo <= hi){
            //被查找的键要么不存在，要么必然存在于a[lo..hi]之中
            int mid = lo + (hi - lo) / 2;
            if(key < a[mid]) hi = mid -1;
            else if(key > a[mid]) lo = mid + 1;
            else return mid;
        }
        return -1;
    }
    public static void main(String[] args){
        int[] whitelist = In.readInts(args[0]);
        Arrays.sort(whitelist);
        while(!StdIn.isEmpty()){
            //读取键值 ，如果不存在于白名单中则将其打印
            int key = StdIn.readInt();
            if(rank(key, whiterlist) < 0)
                StdOut.println(key);
        }
    }
}
```

这段程序接受一个白名单文件（一列整数）作为参数，并会过滤掉标准输入中的所有存在于白名单中的条目，仅将不在白名单上的整数打印到标准输出中。它在rank()静态方法中实现了二分查找算法并高效地完成了这个任务。