# 前言

为了不让自己做公司笔试题时做不出来，所以刷剑指Offer

![](https://img2018.cnblogs.com/blog/1330447/201901/1330447-20190118101708345-636131365.jpg)

所以记录和分享自己的刷题思路和题解写了这个专栏。

## 二叉树的深度

### 题目描述

输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。

### 分析

递归求解：
假如是空节点，则返回0；
否则，原树的深度由左右子树中深度较的深度加1，为原树的深度。

### 贴出代码

```java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
//递归的方式来解
public class Solution1 {
    public int TreeDepth(TreeNode root) {
        if(root == null){
               return 0;
           }
        if(root.left == null & root.right == null){
               return 1;
           }
        int Rleft = TreeDepth(root.left);
        int Rright = TreeDepth(root.right);
        return Rleft > Rright ? Rleft + 1 : Rright + 1;
    }
}
```

## 不用加减乘除做加法

### 题目描述

写一个函数，求两个整数之和，要求在函数体内不得使用+、-、*、/四则运算符号。

### 分析
首先看十进制是如何做的： 5+7=12，三步走 

第一步：相加各位的值，不算进位，得到2。 

第二步：计算进位值，得到10. 如果这一步的进位值为0，那么第一步得到的值就是最终结果。  

第三步：重复上述两步，只是相加的值变成上述两步的得到的结果2和10，得到12。  

同样我们可以用三步走的方式计算二进制值相加： 5-101，7-111 

第一步：相加各位的值，不算进位，得到010，二进制每位相加就相当于各位做异或操作，101^111。  第二步：计算进位值，得到1010，相当于各位做与操作得到101，再向左移一位得到1010，(101&111)<<1。  

第三步重复上述两步， 各位相加 010^1010=1000，进位值为100=(010&1010)<<1。      

继续重复上述两步：1000^100 = 1100，进位值为0，跳出循环，1100为最终结果。  

### 贴出代码

```java
public class Solution {
    public int Add(int num1,int num2) {
        while(nul2 != 0){
            int temp = num1 ^ num2;
            num2 = (num1&num2)<<1;
            num1 = temp;
        }
        return num1;
    }
}
```

## 二叉树的镜像

### 题目描述

操作给定的二叉树，将其变换为源二叉树的镜像。

### 输入描述:

```
二叉树的镜像定义：源二叉树 
    	    8
    	   /  \
    	  6   10
    	 / \  / \
    	5  7 9 11
    	镜像二叉树
    	    8
    	   /  \
    	  10   6
    	 / \  / \
    	11 9 7  5
```

### 分析

二叉树无根或者左右儿子为空不用换。

二叉树的根可以不用换，二叉树根的左儿子和右儿子对换，然后再递归这个操作。

### 贴出代码

```java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public void Mirror(TreeNode root) {
        if(root == null)
            return;
        if(root.left == null && root.right == null)
            return;
         
        TreeNode pTemp = root.left;
        root.left = root.right;
        root.right = pTemp;
         
        if(root.left != null)
            Mirror(root.left);
        if(root.right != null)
            Mirror(root.right);
    }
}
```

## 变态跳台阶

### 题目描述

一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

### 分析
关于本题，前提是n个台阶会有一次n阶的跳法。分析如下:

f(1) = 1
f(2) = f(2-1) + f(2-2)         //f(2-2) 表示2阶一次跳2阶的次数。
f(3) = f(3-1) + f(3-2) + f(3-3) 
...
f(n) = f(n-1) + f(n-2) + f(n-3) + ... + f(n-(n-1)) + f(n-n) 

说明： 
1）这里的f(n) 代表的是n个台阶有一次1,2,...n阶的 跳法数。
2）n = 1时，只有1种跳法，f(1) = 1
1) n = 2时，会有两个跳得方式，一次1阶或者2阶，这回归到了问题（1） ，f(2) = f(2-1) + f(2-2) 
2) n = 3时，会有三种跳得方式，1阶、2阶、3阶，
    那么就是第一次跳出1阶后面剩下：f(3-1);第一次跳出2阶，剩下f(3-2)；第一次3阶，那么剩下f(3-3)
    因此结论是f(3) = f(3-1)+f(3-2)+f(3-3)
3) n = n时，会有n中跳的方式，1阶、2阶...n阶，得出结论：
    f(n) = f(n-1)+f(n-2)+...+f(n-(n-1)) + f(n-n) => f(0) + f(1) + f(2) + f(3) + ... + f(n-1)
4) 由以上已经是一种结论，但是为了简单，我们可以继续简化：
    f(n-1) = f(0) + f(1)+f(2)+f(3) + ... + f((n-1)-1) = f(0) + f(1) + f(2) + f(3) + ... + f(n-2)
    f(n) = f(0) + f(1) + f(2) + f(3) + ... + f(n-2) + f(n-1) = f(n-1) + f(n-1)
    可以得出：
    f(n) = 2*f(n-1)
5) 得出最终结论,在n阶台阶，一次有1、2、...n阶的跳的方式时，总得跳法为：

>​		 | 1       ,(n=0 ) 
>
>f(n) =     | 1       ,(n=1 )
>
>​		| 2*f(n-1),(n>=2)

### 贴出代码
```java
public class Solution {
    public int JumpFloorII(int target) {
        if(target <= 0){
            return -1;
        }
        else if(target == 1){
            return 1;
        }
        else{
            return 2 * JumpFloorII(target - 1);
        }
    }
}
```

## 构建乘积数组

### 题目描述

给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]。不能使用除法。

### 分析

题目翻译过来是这个意思：

> B中i位置的数，是除了A中i位置的数的其他的乘积、


剑指的思路：
B[i]的值可以看作下图的矩阵中每行的乘积。
下三角用连乘可以很容求得，上三角，从下向上也是连乘。
因此我们的思路就很清晰了，先算下三角中的连乘，即我们先算出B[i]中的一部分，然后倒过来按上三角中的分布规律，把另一部分也乘进去。

![](image/35.jpg)

### 贴出代码

```java
import java.util.ArrayList;
public class Solution {
    public int[] multiply(int[] A) {
        int length = A.length;
        int[] B = new int[length];
        if(length != 0 ){
            B[0] = 1;
            //计算下三角连乘
            for(int i = 1; i < length; i++){
                B[i] = B[i-1] * A[i-1];
            }
            int temp = 1;
            //计算上三角
            for(int j = length-2; j >= 0; j--){
                temp *= A[j+1];
                B[j] *= temp;
            }
        }
        return B;
    }
}
```



## 求1+2+3+...+n

### 题目描述

求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。

### 分析

题目要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。所以只能用递归求解。

### 贴出代码

```java
public class Solution {
    public int Sum_Solution(int n) {
        if(n > 0){
            return n + Sum_Solution(n - 1);
        }
        else{
            return 0;
        }
    }
}
```

## 用两个栈实现队列

### 题目描述

用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

### 分析

用一个栈模拟的时候入队出队的情况正好相反，再加一个栈正好符合队列顺序，不过要注意当用第二个栈来模拟出队时，要确保每次只有当第二个栈为空时才从第一个栈中pop值到第二个栈中。

### 贴出代码

```java
public class Solution {
        Stack<Integer> stack1 = new Stack<Integer>();
        Stack<Integer> stack2 = new Stack<Integer>();

        public void push(int node) {
            stack1.push(node);
        }

        public int pop() {
            if(stack1.empty()&&stack2.empty()){
                throw new RuntimeException("Queue is empty");
            }
            if(stack2.empty()){
                while (!stack1.empty()){
                    stack2.push(stack1.pop());
                }
            }
            return stack2.pop();
        }
    }
```

## 连续子数组的最大和

### 题目描述

HZ偶尔会拿些专业问题来忽悠那些非计算机专业的同学。今天测试组开完会后,他又发话了:在古老的一维模式识别中,常常需要计算连续子向量的最大和,当向量全为正数的时候,问题很好解决。但是,如果向量中包含负数,是否应该包含某个负数,并期望旁边的正数会弥补它呢？例如:{6,-3,-2,7,-15,1,2,2},连续子向量的最大和为8(从第0个开始,到第3个为止)。给一个数组，返回它的最大连续子序列的和，你会不会被他忽悠住？(子向量的长度至少是1)

### 分析

最大子序列是要找出由数组成的一维数组中和最大的连续子序列。比如{5,-3,4,2}的最大子序列就是 {5,-3,4,2}，它的和是8,达到最大；而 {5,-6,4,2}的最大子序列是{4,2}，它的和是6。你已经看出来了，找最大子序列的方法很简单，只要前i项的和还没有小于0那么子序列就一直向后扩展，否则丢弃之前的子序列开始新的子序列，同时我们要记下各个子序列的和，最后找到和最大的子序列。
动态方程： 
最大子序列和是连续的子序列 
ThisSum[i]表示第i处，以A[i]结尾的子序列的最大和。 
则状态方程为ThisSum[i]=max(ThisSum[i-1]+nums[i],nums[i])
也即是说，如果ThisSum[i-1]<0则ThisSum[i]就是nums[i],否则ThisSum[i]=nums[i]+ThisSum[i-1]（因为是以i结尾的

### 贴出代码

```java
public class Solution {
    public int FindGreatestSumOfSubArray(int[] array) {
        int r,l;
        int maxSum = array[0],thisSum = array[0];
        int new_l = 0;
        for(int i = 1;i < array.length; i++){
            if(thisSum < 0){
                thisSum = array[i];
                new_l = i;
            }else {
                thisSum += array[i];
            }
            if(thisSum > maxSum){
                maxSum = thisSum;
                l = new_l;
                r = i;
            }
        }
        return maxSum;
    }
}
```

## 矩形覆盖

### 题目描述

我们可以用2\*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2\*1的小矩形无重叠地覆盖一个2\*n的大矩形，总共有多少种方法？

### 分析

这题看讨论的我不是很会。

  依旧是斐波那契数列 

  2\*n的大矩形，和n个2\*1的小矩形 

  其中target*2为大矩阵的大小 

  有以下几种情形： 

1. target <= 0 大矩形为<= 2*0,直接return 1； 

2. target = 1大矩形为2*1，只有一种摆放方法，return1； 

3. target = 2 大矩形为2*2，有两种摆放方法，return2； 

4. target = n 分为两步考虑： 

​          第一次摆放一块 2*1 的小矩阵，则摆放方法总共为f(target - 1)  

| √    |      |      |      |      |      |      |      |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| √    |      |      |      |      |      |      |      |

第一次摆放一块1*2的小矩阵，则摆放方法总共为f(target-2) 

  因为，摆放了一块1*2的小矩阵（用√√表示），对应下方的1*2（用××表示）摆放方法就确定了，所以为f(targte-2) 

| √    | √    |      |      |      |      |      |      |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| ×    | ×    |      |      |      |      |      |      |

### 贴出代码

```java
public class Solution {
    public int RectCover(int target) {
      if(target  <= 0){
            return 0;
        }else if(target <= 2){
            return target;
        }else{
            return RectCover((target-1))+RectCover(target-2);
        }
    }
}
```

## 二进制中1的个数

### 题目描述

输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

### 分析
用1（1自身左移运算，其实后来就不是1了）和n的每位进行位与，来判断1的个数
### 贴出代码
```java
public class Solution {
    public int NumberOf1(int n) {
        int count = 0;
        while(n != 0){
            ++count;
            n = (n -1) & n;
        }
        return count;
    }
}
```

## 跳台阶
### 题目描述
一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。
### 分析
对于本题,前提只有 一次 1阶或者2阶的跳法。

a.如果两种跳法，1阶或者2阶，那么假定第一次跳的是一阶，那么剩下的是n-1个台阶，跳法是f(n-1);
b.假定第一次跳的是2阶，那么剩下的是n-2个台阶，跳法是f(n-2)
c.由a\b假设可以得出总跳法为: f(n) = f(n-1) + f(n-2) 
d.然后通过实际的情况可以得出：只有一阶的时候 f(1) = 1 ,只有两阶的时候可以有 f(2) = 2
e.可以发现最终得出的是一个斐波那契数列：

              | 1, (n=1)

f(n) =     | 2, (n=2)

              | f(n-1)+f(n-2) ,(n>2,n为整数)
### 贴出代码
```java
public class Solution {
    public int JumpFloor(int target) {
        if(target <= 0){
            return -1;
        }else if(target == 1){
            return 1;
        }else if(target == 2){
            return 2;
        }else{
            return JumpFloor(target - 1) + JumpFloor(target -2);
        }
    }
}
```
## 平衡二叉树
### 题目描述
输入一棵二叉树，判断该二叉树是否是平衡二叉树。
### 分析
从下往上遍历，如果子树是平衡二叉树，则返回子树的高度；如果发现子树不是平衡二叉树，则直接停止遍历，这样至多只对每个结点访问一次。
### 贴出代码
```java
public class Solution {
    public boolean IsBalanced_Solution(TreeNode root) {
        return getDepth(root) != -1;
    }
    
    private int getDepth(TreeNode root) {
        if (root == null) return 0;
        int left = getDepth(root.left);
        if (left == -1) return -1;
        int right = getDepth(root.right);
        if (right == -1) return -1;
        return Math.abs(left - right) > 1 ? -1 : 1 + Math.max(left, right);
    }
}
```
## 整数中1出现的次数（从1到n整数中1出现的次数）
### 题目描述
求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从1 到 n 中1出现的次数）。
### 分析
设N = abcde ,其中abcde分别为十进制中各位上的数字。
如果要计算百位上1出现的次数，它要受到3方面的影响：百位上的数字，百位以下（低位）的数字，百位以上（高位）的数字。
① 如果百位上数字为0，百位上可能出现1的次数由更高位决定。比如：12013，则可以知道百位出现1的情况可能是：100~199，1100~1199,2100~2199，，...，11100~11199，一共1200个。可以看出是由更高位数字（12）决定，并且等于更高位数字（12）乘以 当前位数（100）。
② 如果百位上数字为1，百位上可能出现1的次数不仅受更高位影响还受低位影响。比如：12113，则可以知道百位受高位影响出现的情况是：100~199，1100~1199,2100~2199，，....，11100~11199，一共1200个。和上面情况一样，并且等于更高位数字（12）乘以 当前位数（100）。但同时它还受低位影响，百位出现1的情况是：12100~12113,一共114个，等于低位数字（113）+1。
③ 如果百位上数字大于1（2~9），则百位上出现1的情况仅由更高位决定，比如12213，则百位出现1的情况是：100~199,1100~1199，2100~2199，...，11100~11199,12100~12199,一共有1300个，并且等于更高位数字+1（12+1）乘以当前位数（100）。
### 贴出代码
```java
public class Solution {
    public int NumberOf1Between1AndN_Solution(int n) {
        int count = 0;//1的个数
        int i = 1;//当前位
        int current = 0,after = 0,before = 0;
        while((n/i)!= 0){           
            current = (n/i)%10; //高位数字
            before = n/(i*10); //当前位数字
            after = n-(n/i)*i; //低位数字
            //如果为0,出现1的次数由高位决定,等于高位数字 * 当前位数
            if (current == 0)
                count += before*i;
            //如果为1,出现1的次数由高位和低位决定,高位*当前位+低位+1
            else if(current == 1)
                count += before * i + after + 1;
            //如果大于1,出现1的次数由高位决定,//（高位数字+1）* 当前位数
            else{
                count += (before + 1) * i;
            }    
            //前移一位
            i = i*10;
        }
        return count;
    }
}
```
## 两个链表的第一个公共结点
### 题目描述
输入两个链表，找出它们的第一个公共结点。
### 分析
利用HashMap的特性。
### 贴出代码
```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
import java.util.HashMap;
public class Solution {
    public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
        ListNode current1 = pHead1;
        ListNode current2 = pHead2;
 
 
        HashMap<ListNode, Integer> hashMap = new HashMap<ListNode, Integer>();
        while (current1 != null) {
            hashMap.put(current1, null);
            current1 = current1.next;
        }
        while (current2 != null) {
            if (hashMap.containsKey(current2))
                return current2;
            current2 = current2.next;
        }
 
        return null;
 
    }
}
```
## 旋转数组的最小数字
### 题目描述
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
### 分析
采用二分法解答这个问题，
mid = low + (high - low)/2
需要考虑三种情况：
(1)array[mid] > array[high]:
出现这种情况的array类似[3,4,5,6,0,1,2]，此时最小数字一定在mid的右边。
low = mid + 1
(2)array[mid] == array[high]:
出现这种情况的array类似 [1,0,1,1,1] 或者[1,1,1,0,1]，此时最小数字不好判断在mid左边
还是右边,这时只好一个一个试 ，
high = high - 1
(3)array[mid] < array[high]:
出现这种情况的array类似[2,2,3,4,5,6,6],此时最小数字一定就是array[mid]或者在mid的左
边。因为右边必然都是递增的。
high = mid
注意这里有个坑：如果待查询的范围最后只剩两个数，那么mid 一定会指向下标靠前的数字
比如 array = [4,6]
array[low] = 4 ;array[mid] = 4 ; array[high] = 6 ;
如果high = mid - 1，就会产生错误， 因此high = mid
但情形(1)中low = mid + 1就不会错误
### 贴出代码
```java
import java.util.ArrayList;
public class Solution {
    public int minNumberInRotateArray(int [] array) {
        int low = 0;
        int high = array.length - 1;
        while(low < high){
            int mid = low + (high - low) / 2;
            if(array[mid] > array[high]){
                low = mid + 1;
            }else if(array[mid] == array[high]){
                high = high - 1;
            }else{
                high = mid;
            }
        }
        return array[low];
    }
}
```
## 数值的整数次方
### 题目描述
给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
### 分析
递归：n为偶数，a^n=a^n/2*a^n/2;n为奇数，a^n=（a^（n-1）/2）*（a^（n-1/2））*a
### 贴出代码
```java
public class Solution {
    public double Power(double base, int exponent) {
        int n = Math.abs(exponent);
        if(n == 0){
            return 1;
        }
        if(n == 1){
            return base;
        }
        double result = Power(base,n>>1);
        result *= result;
        if((n&1) == 1){
            result *= base;
        }
        if(exponent < 0){
            result = 1/result;
        }
        return result;
  }
}
```
## 包含min函数的栈
### 题目描述
定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。
###分析

### 贴出代码
```java
import java.util.Stack;
import java.util.Arrays;

public class Solution {

    private int size;
    private int min = Integer.MAX_VALUE;
    private Stack<Integer> minStack = new Stack<Integer>();
    private Integer[] elements = new Integer[10];

    public void push(int node) {
        ensureCapacity(size+1);
        elements[size++] = node;
        if (node <= min){
            minStack.push(node);
            min = minStack.peek();
        }else {
            minStack.push(min);
        }
    }

    private void ensureCapacity(int size ){
        int len = elements.length;
        if(size > len){
            int newLen = (len * 3) / 2 + 1;
            elements = Arrays.copyOf(elements,newLen);
        }
    }

    public void pop() {
        Integer top = top();
        if(top != null){
            elements[size - 1] = (Integer) null;
        }
        size--;
        minStack.pop();
        min = minStack.peek();
    }

    public int top() {
        if(!empty()){
            if(size - 1 >= 0){
                return  elements[size - 1];
            }
        }
        return (Integer) null;
    }

    public boolean empty(){
        return size == 0;
    }

    public int min() {
        return min;
    }
}
```
## 字符流中第一个不重复的字符
### 题目描述
请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。
输出描述：
> 如果当前字符流没有存在出现一次的字符，返回#字符。
###分析
一个字符占8位，因此不会超过256个，可以申请一个256大小的数组来实现一个简易的哈希表。时间复杂度为O(n)，空间复杂度O(n).
### 贴出代码
```java
public class Solution {
    int[] hashtable = new int[256];
    StringBuffer s = new StringBuffer();
    //Insert one char from stringstream
    public void Insert(char ch)
    {
        s.append(ch);
        if(hashtable[ch] == 0){
            hashtable[ch] = 1;
        }else{
            hashtable[ch] += 1;
        }
    }
  //return the first appearence once char in current stringstream
    public char FirstAppearingOnce()
    {
        char[] str = s.toString().toCharArray();
        for(char c : str){
            if(hashtable[c] == 1){
                return c;
            }
        }
        return '#';
    }
}
```
## 链表中环的入口结点
### 题目描述
给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。
###分析
如果链表中环 有n个结点，指针P1在链表上向前移动n步，然后两个指针以相同的速度向前移动。
当第二个指针指向环的入口结点时，第一个指针已经围绕着环走了一圈又回到了入口结点。
所以首先要得到环中结点的数目。
### 贴出代码
```java
/*
 public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}
*/

public class Solution {
    
    public static ListNode meetingNode(ListNode head){
        if(head == null){
            return null;
        }

        ListNode slow = head.next;
        if(slow == null){
            return null;
        }

        ListNode fast = slow.next;
        while(slow != null && fast != null){
            if(slow == fast){
                return fast;
            }
            slow = slow.next;
            fast = fast.next;

            if(fast != slow){
                fast = fast.next;
            }
        }
        return null;
    }

    public ListNode EntryNodeOfLoop(ListNode pHead)
    {
        ListNode meetingNode = meetingNode(pHead);
        if(meetingNode == null){
            return null;
        }

        int nodesInLoop = 1;
        ListNode p1 = meetingNode;
        while (p1.next != meetingNode){
            p1 = p1.next;
            ++nodesInLoop;
        }

        p1 = pHead;
        for (int i = 0; i < nodesInLoop; i++){
            p1 = p1.next;
        }

        ListNode p2 = pHead;
        while (p1 != p2){
            p1 = p1.next;
            p2 = p2.next;
        }
        return p1;
    }
}
```
## 左旋转字符串
### 题目描述
汇编语言中有一种移位指令叫做循环左移（ROL），现在有个简单的任务，就是用字符串模拟这个指令的运算结果。对于一个给定的字符序列S，请你把其循环左移K位后的序列输出。例如，字符序列S=”abcXYZdef”,要求输出循环左移3位后的结果，即“XYZdefabc”。是不是很简单？OK，搞定它！
###分析
看题。
### 贴出代码
```java
public class Solution {
    public String LeftRotateString(String str,int n) {
        if(str == null || n < 0 || n > str.length()){
            return "";
        }
        StringBuffer sb = new StringBuffer(str);
        sb.append(sb.substring(0, n));
        return sb.substring(n, sb.length());
    }
}
```
## 对称的二叉树
### 题目描述
请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。
###分析
首先根节点以及其左右子树，左子树的左子树和右子树的右子树相同左子树的右子树和右子树的左子树相同即可，采用递归。
### 贴出代码
```java
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    boolean isSymmetrical(TreeNode pRoot)
    {
        if(pRoot == null){
            return true;
        }
        return comRoot(pRoot.left,pRoot.right);
    }

    boolean comRoot(TreeNode left, TreeNode right){
        if(left == null){
            return right == null;
        }
        if(right == null){
            return false;
        }
        if(left.val != right.val){
            return false;
        }
        return comRoot(left.right, right.left) && comRoot(left.left, right.right);
    }
}
```

## 把二叉树打印成多行
### 题目描述
从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。
### 分析
利用递归的方法进行先序遍历，传递深度，递归深入一层扩容一层数组，先序遍历又保证了同层节点按从左到右入数组
### 贴出代码
```java
import java.util.ArrayList;


/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> list = new ArrayList<>();
        depth(pRoot,1,list);
        return list;
    }
    
    public void depth(TreeNode root, int depth, ArrayList<ArrayList<Integer>> list){
        if(root == null){
            return;
        }
        if(depth > list.size()){
            list.add(new ArrayList<Integer>());
        }
        list.get(depth - 1).add(root.val);
        
        depth(root.left, depth + 1, list);
        depth(root.right, depth + 1, list);
    }
    
}
```

## 孩子们的游戏(圆圈中最后剩下的数)
### 题目描述
每年六一儿童节,牛客都会准备一些小礼物去看望孤儿院的小朋友,今年亦是如此。HF作为牛客的资深元老,自然也准备了一些小游戏。其中,有个游戏是这样的:首先,让小朋友们围成一个大圈。然后,他随机指定一个数m,让编号为0的小朋友开始报数。每次喊到m-1的那个小朋友要出列唱首歌,然后可以在礼品箱中任意的挑选礼物,并且不再回到圈中,从他的下一个小朋友开始,继续0...m-1报数....这样下去....直到剩下最后一个小朋友,可以不用表演,并且拿到牛客名贵的“名侦探柯南”典藏版(名额有限哦!!^_^)。请你试着想下,哪个小朋友会得到这份礼品呢？(注：小朋友的编号是从0到n-1)
### 分析
题目大概是让我们返回得到柯南的小朋友序号，所以，将小朋友序号存入ArrayList中，在根据坐标(m - 1)来移除唱歌挑选礼物的小朋友，最终一个循环就可以找到我们要找的小朋友
### 贴出代码
```java
import java.util.ArrayList;
public class Solution {
    public int LastRemaining_Solution(int n, int m) {
        if(m == 0 || n == 0){
            return -1;
        }
        ArrayList<Integer> data = new ArrayList<Integer>();
        for(int i = 0; i < n ; i++){
            data.add(i);
        }
        int index = -1;
        
        while(data.size() > 1){
            index = (index + m) % data.size(); // 此时index 等于 arraylist中的 m - 1位置
            data.remove(index); // 移除唱歌的孩子
            index--; // 因为原数组减去一个元素，坐标也相应减去1
        }
        return data.get(0); // 获得剩下的那个人就是能得到”名侦探柯南”典藏版
    }
}
```
## 数字在排序数组中出现的次数
### 题目描述
统计一个数字在排序数组中出现的次数。
### 分析
因为是一个排序数组，所以利用二分查找，找出第一个k值和最后一个k值，再相减差值+1就是出现的次数。
### 贴出代码
```java
public class Solution {
    public int GetNumberOfK(int [] array , int k) {
        if(array == null || array.length == 0){
            return 0;
        }
        int first = getFirstK(array, k, 0, array.length - 1);
        int last = getLastK(array, k, 0, array.length - 1);
        if(first == -1 || last == -1){
            return 0;
        }
        else{
            return last - first + 1;
        }
    }
    
    public int getFirstK(int[] array, int k, int start, int end){
        while(start <= end){
            int mid = (start + end) / 2;
            if(k < array[mid]){
                end = mid - 1;
            }else if(k > array[mid]){
                start = mid + 1;
            }
            else{
                if((mid > 0 && array[mid - 1] != k) || mid == 0){
                    return mid;
                }else{
                    end = mid - 1;
                }
            }
        }
        return -1;
    }
    
    public int getLastK(int[] array, int k, int start, int end){
        while(start <= end){
            int mid = (start + end) / 2;
            if(k < array[mid]){
                end = mid - 1;
            }else if(k > array[mid]){
                start = mid + 1;
            }
            else{
                if((mid < array.length - 1 && array[mid + 1] != k) || mid == array.length -1){
                    return mid;
                }
                else{
                    start = mid + 1;
                }
            }
        }
        return -1;
    }
}
```
## 二叉树的下一个结点
### 题目描述
给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。
### 分析
分析二叉树的下一个节点，一共有以下情况：
1.二叉树为空，则返回空；
2.节点右孩子存在，则设置一个指针从该节点的右孩子出发，一直沿着指向左子结点的指针找到的叶子节点即为下一个节点；
3.节点不是根节点。如果该节点是其父节点的左孩子，则返回父节点；否则继续向上遍历其父节点的父节点，重复之前的判断，返回结果。
### 贴出代码
```java
/*
public class TreeLinkNode {
    int val;
    TreeLinkNode left = null;
    TreeLinkNode right = null;
    TreeLinkNode next = null;

    TreeLinkNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    public TreeLinkNode GetNext(TreeLinkNode pNode)
    {
        if(pNode == null){
            return pNode;
        }
        if(pNode.right != null){
            pNode = pNode.right;
            while(pNode.left != null){
                pNode = pNode.left;
            }
            return pNode;
        }else if(pNode.next != null && pNode.next.left == pNode){
            return pNode.next;
        }else if(pNode.next != null && pNode.next.right == pNode){
            while(pNode.next != null && pNode.next.left != pNode){
                pNode = pNode.next;
            }
            return pNode.next;
        }else{
            return pNode.next;
        }
    }
}
```
## 数组中只出现一次的数字
### 题目描述
一个整型数组里除了两个数字之外，其他的数字都出现了偶数次。请写程序找出这两个只出现一次的数字。
### 分析
先定义一个存两个数字的数组，先遍历array数组，找到不包含的两个数字，通过contains()来判断是否含有，含有则remove()。
### 贴出代码
```java
//num1,num2分别为长度为1的数组。传出参数
//将num1[0],num2[0]设置为返回结果
import java.util.ArrayList;
import java.util.List;
public class Solution {
    public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < array.length; i++){
            if(!list.contains(String.valueOf(array[i]))){
                list.add(String.valueOf(array[i]));
            }else {
                list.remove(String.valueOf(array[i]));
            }
        }
        if (list.size() != 0){
            num1[0] = Integer.valueOf(list.get(0));
            num2[0] = Integer.valueOf(list.get(1));
        }
    }
}
```

## 斐波那契数列
### 题目描述
大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项（从0开始，第0项为0）。
n<=39
### 分析
斐波那契数列：
> F[n] = F[n - 1] + F[n -2]
>
采用尾递归，防止递归时栈溢出。
### 贴出代码
```java
public class Solution {
    public int Fibonacci(int n) {
        return Fibonacci(n,0,1);
    }
    
    private static int Fibonacci(int n,int acc1,int acc2){
        if(n==0) return 0;
        if(n==1) return acc2;
        else  
            return Fibonacci(n - 1, acc2, acc1 + acc2);
    }
}
```

## 和为S的两个数字
### 题目描述
输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。

**输出描述:**
> 对应每个测试案例，输出两个数，小的先输出。
### 分析
由于是排序好的数组，所以左右逼近。
### 贴出代码
```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> FindNumbersWithSum(int [] array,int sum) {
        ArrayList<Integer> list = new ArrayList<>();
        if (array == null || array.length < 2){
            return list;
        }
        int i = 0, j = array.length - 1;
        while (i < j){
            if (array[i] + array[j] == sum){
                list.add(array[i]);
                list.add(array[j]);
                return list;
            }else if (array[i] + array[j] > sum){
                j --;
            }else {
                i ++;
            }
        }
        return list;
    }
}
```

## 反转链表
### 题目描述
输入一个链表，反转链表后，输出新链表的表头。
### 分析
单链表的反转。
### 贴出代码
```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode ReverseList(ListNode head) {
        if (head == null){
            return null;
        }
        ListNode pre = null;
        ListNode next = null;
        while (head != null){
            next = head.next;
            head.next = pre;
            pre = head;
            head = next;
        }
        return pre;
    }
}
```

## 栈的压入、弹出序列
### 题目描述
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
### 分析
借用一个辅助的栈，遍历压栈顺序，先讲第一个放入栈中，这里是1，然后判断栈顶元素是不是出栈顺序的第一个元素，这里是4，很显然1≠4，所以我们继续压栈，直到相等以后开始出栈，出栈一个元素，则将出栈顺序向后移动一位，直到不相等，这样循环等压栈顺序遍历完成，如果辅助栈还不为空，说明弹出序列不是该栈的弹出顺序。

举例：

入栈1,2,3,4,5

出栈4,5,3,2,1

首先1入辅助栈，此时栈顶1≠4，继续入栈2

此时栈顶2≠4，继续入栈3

此时栈顶3≠4，继续入栈4

此时栈顶4＝4，出栈4，弹出序列向后一位，此时为5，,辅助栈里面是1,2,3

此时栈顶3≠5，继续入栈5

此时栈顶5=5，出栈5,弹出序列向后一位，此时为3，,辅助栈里面是1,2,3

….

依次执行，最后辅助栈为空。如果不为空说明弹出序列不是该栈的弹出顺序。
### 贴出代码
```java
import java.util.ArrayList;
import java.util.Stack;
public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
         if (pushA.length == 0 || popA.length == 0){
            return false;
        }
        Stack<Integer> s = new Stack<>();
        int popIndex = 0;
        for (int i = 0; i < pushA.length; i++){
            s.push(pushA[i]);
            while (!s.empty() && s.peek() == popA[popIndex]){
                s.pop();
                popIndex ++;
            }
        }
        return s.empty();
    }
}
```

## 数组中重复的数字
### 题目描述
在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。
### 分析
简单的包含问题。
### 贴出代码
```java
public class Solution {
    // Parameters:
    //    numbers:     an array of integers
    //    length:      the length of array numbers
    //    duplication: (Output) the duplicated number in the array number,length of duplication array is 1,so using duplication[0] = ? in implementation;
    //                  Here duplication like pointor in C/C++, duplication[0] equal *duplication in C/C++
    //    这里要特别注意~返回任意重复的一个，赋值duplication[0]
    // Return value:       true if the input is valid, and there are some duplications in the array number
    //                     otherwise false
    public boolean duplicate(int numbers[],int length,int [] duplication) {
        boolean[] k = new boolean[length];
		for(int i = 0; i < k.length; i++) {
			if(k[numbers[i]] == true) {
				duplication[0] = numbers[i];
				return true;
			}
			k[numbers[i]] = true;
		}
		return false;
    }
}
```

## 二叉搜索树与双向链表
### 题目描述
输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。
### 分析
直接用中序遍历
### 贴出代码
```java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    TreeNode head = null;
	TreeNode realHead = null;
	
	public TreeNode Convert(TreeNode pRootOfTree) {
        ConvertSub(pRootOfTree);
        return realHead;
    }
	
	private void ConvertSub(TreeNode pRootOfTree) {
		if(pRootOfTree == null) {
			return;
		}
		ConvertSub(pRootOfTree.left);
		if(head == null) {
			head = pRootOfTree;
			realHead = pRootOfTree;
		}else {
			head.right = pRootOfTree;
			pRootOfTree.left = head;
			head = pRootOfTree;
		}
		ConvertSub(pRootOfTree.right);
	}
}
```

## 合并两个排序的链表
### 题目描述
输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。
### 分析
因为是递增的两个链表，为空则抛出另一个数组，如果不为空，谁小谁在前面，next指向下一个节点。
### 贴出代码
```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
        if(list1 == null) {
        	return list2;
        }else if(list2 == null) {
        	return list1;
        }
        if(list1.val <= list2.val) {
        	list1.next = Merge(list1.next, list2);
        	return list1;
        }else {
        	list2.next = Merge(list1, list2.next);
        	return list2;
        }
    }
}
```

## 数组中出现次数超过一半的数字
### 题目描述
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。
### 分析
先将数组进行排序，那么数组中肯定会有一个数超过了数组的一般，如果相等则+1，然后判断count是否大于数组长度的一半。
### 贴出代码
```java
import java.util.Arrays;
public class Solution {
    public int MoreThanHalfNum_Solution(int [] array) {
        Arrays.sort(array);
        int count = 0;
        for(int i = 0; i < array.length; i++) {
        	if(array[i] == array[array.length / 2]) {
        		count++;
        	}
        }
        if(count > array.length / 2) {
        	return array[array.length / 2];
        }else {
        	return 0;
        }
    }
}
```

## 把数组排成最小的数
### 题目描述
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。
### 分析

### 贴出代码
```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.Collections;
public class Solution {
    public String PrintMinNumber(int [] numbers) {
        int n;
		String s = "";
		ArrayList<Integer> list = new ArrayList<Integer>();
		n = numbers.length;
		for(int i = 0; i < n; i++) {
			list.add(numbers[i]);
		}
		Collections.sort(list, new Comparator<Integer>() {
			public int compare(Integer str1, Integer str2) {
				String s1 = str1 + "" + str2;
				String s2 = str2 + "" + str1;
				return s1.compareTo(s2);
			}
		});
		for(int j : list) {
			s += j;
		}
		return s;
    }
}
```
## 从上往下打印二叉树
### 题目描述
从上往下打印出二叉树的每个节点，同层节点从左至右打印。
### 分析
用arraylist模拟一个队列来存储相应的TreeNode
### 贴出代码
```java
import java.util.ArrayList;
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> list = new ArrayList<>();
		ArrayList<TreeNode> queue = new ArrayList<>();
		if(root == null) {
			return list;
		}
		queue.add(root);
		while(queue.size() != 0) {
			TreeNode temp = queue.remove(0);
			if(temp.left != null) {
				queue.add(temp.left);
			}
			if(temp.right != null) {
				queue.add(temp.right);
			}
			list.add(temp.val);
		}
		return list;
    }
}
```

## 表示数值的字符串
### 题目描述
请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。
### 分析
以下对正则进行解释:
	[\\+\\-]?            -> 正或负符号出现与否
	\\d*                 -> 整数部分是否出现，如-.34 或 +3.34均符合
	(\\.\\d+)?           -> 如果出现小数点，那么小数点后面必须有数字；否则一起不出现
	([eE][\\+\\-]?\\d+)? -> 如果存在指数部分，那么e或E肯定出现，+或-可以不出现，紧接着必须跟着整数；或者整个部分都不出现
### 贴出代码
```java
public class Solution {
    public boolean isNumeric(char[] str) {
        String string = String.valueOf(str);
        return string.matches("[\\+\\-]?\\d*(\\.\\d+)?([eE][\\+\\-]?\\d+)?");
    }
}
```

## 和为S的连续正数序列
### 题目描述
小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck!
**输出描述:**
> 输出所有和为S的连续正数序列。序列内按照从小至大的顺序，序列间按照开始数字从小到大的顺序

### 分析
双指针技术，就是相当于有一个窗口，窗口的左右两边就是两个指针，我们根据窗口内值之和来确定窗口的位置和宽度。非常牛逼的思路，虽然双指针或者所谓的滑动窗口技巧还是蛮常见的，但是这一题还真想不到这个思路。
### 贴出代码
```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
       ArrayList<ArrayList<Integer>> result = new ArrayList<>();
	       int plow = 1, phigh = 2;
	       while (phigh > plow) {
	    	   int cur = (phigh + plow) * (phigh - plow + 1) / 2;
	    	   if(cur == sum) {
	    		   ArrayList<Integer> list = new ArrayList<>();
	    		   for(int i = plow; i <= phigh; i++) {
	    			   list.add(i);
	    		   }
	    		   result.add(list);
	    		   plow++;
	    	   }else if(cur < sum) {
	    		   phigh ++;
	    	   }else {
	    		   plow++;
	    	   }
	       }
	       return result;
    }
}
```

## 把字符串转换成整数
### 题目描述
将一个字符串转换成一个整数(实现Integer.valueOf(string)的功能，但是string不符合数字要求时返回0)，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0。
**输入描述:**
> 输入一个字符串,包括数字字母符号,可以为空

**输出描述:**
> 如果是合法的数值表达则返回该数字，否则返回0

**示例1**
> 输入
> +2147483647
>    1a33

> 输出
> 2147483647
>    0
### 分析

### 贴出代码
```java
public class Solution {
    public int StrToInt(String str) {
        if(str.equals("") || str.length() == 0) {
			return 0;
		}
		char[] a = str.toCharArray();
		int fuhao = 0;
		if (a[0] == '-') {
			fuhao = 1;
		}
		int sum = 0;
		for(int i = fuhao; i < a.length; i++) {
			if(a[i] == '+') {
				continue;
			}
			if(a[i] < 48 || a[i] > 57) {
				return 0;
			}
			sum = sum * 10 + a[i] -48;
		}
		return fuhao == 0 ? sum : sum * -1;
    }
}
```

## 二叉树中和为某一值的路径
### 题目描述
输入一颗二叉树的跟节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)
### 分析
递归遍历即可。
### 贴出代码
```java
import java.util.ArrayList;
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;
    }
}
*/
public class Solution {
    private ArrayList<ArrayList<Integer>> listAll = new ArrayList<ArrayList<Integer>>();
	private ArrayList<Integer> list = new ArrayList<Integer>();
	public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
        if(root == null) {
        	return listAll;
        }
        list.add(root.val);
        target -= root.val;
        if(target == 0 && root.left == null && root.right == null) {
        	listAll.add(new ArrayList<Integer>(list));
        }
        FindPath(root.left, target);
        FindPath(root.right, target);
        list.remove(list.size() - 1);
        return listAll;
    }
}
```
## 第一个只出现一次的字符
### 题目描述
在一个字符串(0<=字符串长度<=10000，全部由字母组成)中找到第一个只出现一次的字符,并返回它的位置, 如果没有则返回 -1（需要区分大小写）.
### 分析

### 贴出代码
```java
import java.util.HashMap;
public class Solution {
    public int FirstNotRepeatingChar(String str) {
        if (str == null){
	        return -1;
        }
        HashMap<Character, Integer> map = new HashMap<>();
        for (int i = 0; i < str.length(); i++){
            if (map.containsKey(str.charAt(i))){
                int time = map.get(str.charAt(i));
                map.put(str.charAt(i),++time);
            }
            else{
                map.put(str.charAt(i),1);
            }
        }
        int i = 0;
        for (; i <str.length(); i++){
            if (map.get(str.charAt(i)) == 1){
                return i;
            }
        }
        return -1;
    }
}
```

## 数据流中的中位数
### 题目描述
如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。
### 分析

### 贴出代码
```java
import java.util.Comparator;
import java.util.PriorityQueue;
public class Solution {

    private int count = 0;
    // 小根堆
    private PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    // 大根堆
    private PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>(15, new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o2 - o1;
        }
    });

    public void Insert(Integer num) {

        if (count  % 2 == 0){
            maxHeap.offer(num);
            int filteredMaxNum = maxHeap.poll();
            minHeap.offer(filteredMaxNum);
        }else{
            minHeap.offer(num);
            int filteredMinNum = minHeap.poll();
            maxHeap.offer(filteredMinNum);
        }
        count++;
    }

    public Double GetMedian() {
        if (count  % 2 == 0) {
            return new Double((minHeap.peek() + maxHeap.peek())) / 2;
        }else {
            return new Double(minHeap.peek());
        }
    }


}
```

## 调整数组顺序使奇数位于偶数前面
### 题目描述
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。
### 分析

### 贴出代码
```java
public class Solution {
    public void reOrderArray(int [] array) {
        if (array == null || array.length == 0){
            return;
        }
        int i = 0,j;
        while(i < array.length){
            while (i < array.length && !isEven(array[i])){
                i++;
            }
            j = i+1;
            while (j < array.length && isEven(array[j])){
                j++;
            }
            if (j < array.length){
                int tmp = array[j];
                for (int j2 = j -1 ; j2 >= i ; j2--){
                    array[j2+1] = array[j2];
                }
                array[i++] = tmp;
            }else {
                break;
            }
        }
    }

    private boolean isEven(int n){
        if (n % 2 == 0){
            return true;
        }
        return false;
    }
}
```

## 扑克牌顺子
### 题目描述
LL今天心情特别好,因为他去买了一副扑克牌,发现里面居然有2个大王,2个小王(一副牌原本是54张^_^)...他随机从中抽出了5张牌,想测测自己的手气,看看能不能抽到顺子,如果抽到的话,他决定去买体育彩票,嘿嘿！！“红心A,黑桃3,小王,大王,方片5”,“Oh My God!”不是顺子.....LL不高兴了,他想了想,决定大\小 王可以看成任何数字,并且A看作1,J为11,Q为12,K为13。上面的5张牌就可以变成“1,2,3,4,5”(大小王分别看作2和4),“So Lucky!”。LL决定去买体育彩票啦。 现在,要求你使用这幅牌模拟上面的过程,然后告诉我们LL的运气如何， 如果牌能组成顺子就输出true，否则就输出false。为了方便起见,你可以认为大小王是0。
### 分析

### 贴出代码
```java
public class Solution {
    public boolean isContinuous(int [] numbers) {
       int[] d = new int[14];
       d[0] = -5;
       int len = numbers.length;
       int max = -1;
       int min = 14;
       for (int i = 0 ; i < len; i++){
           d[numbers[i]] ++;
           // 是0就继续
           if (numbers[i] == 0){
               continue;
           }
           if (d[numbers[i]] > 1){
               // 出现了两次
               return false;
           }
           if (numbers[i] > max){
               // 值比最大值还大
               max = numbers[i];
           }
           if (numbers[i] < min){
               // 值比最小值还小
               min = numbers[i];
           }
       }
       if (max - min < 5){
           return true;

       }
       return false;
    }
}
```

## 替换空格
### 题目描述
请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。
### 分析

### 贴出代码
```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
    	String sti = str.toString();
        char[] chars = sti.toCharArray();
        StringBuffer out = new StringBuffer();
        for (int i = 0 ; i < chars.length; i++){
            if (chars[i] == ' '){
                out.append("%20");
            }else {
                out.append(chars[i]);
            }
        }
        return out.toString();
    }
}
```

## 从尾到头打印链表
### 题目描述
输入一个链表，按链表值从尾到头的顺序返回一个ArrayList。
### 分析
递归即可。
### 贴出代码
```java
/**
*    public class ListNode {
*        int val;
*        ListNode next = null;
*
*        ListNode(int val) {
*            this.val = val;
*        }
*    }
*
*/
import java.util.ArrayList;
public class Solution {
    ArrayList<Integer> arrayList = new ArrayList<Integer>();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if (listNode != null){
            this.printListFromTailToHead(listNode.next);
            arrayList.add(listNode.val);
        }
        return arrayList;
    }
}
```

## 二叉搜索树的第k个结点
### 题目描述
给定一棵二叉搜索树，请找出其中的第k小的结点。例如，（5，3，7，2，4，6，8）中，按结点数值大小顺序第三小结点的值为4。
### 分析

### 贴出代码
```java
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    int index = 0;
    TreeNode KthNode(TreeNode pRoot, int k)
    {
        if(pRoot != null){
            TreeNode node = KthNode(pRoot.left,k);
            if(node != null){
                return node;
            }
            index++;
            if(index == k){
                return pRoot;
            }
            node = KthNode(pRoot.right,k);
            if(node != null){
                return node;
            }
            
        }
        return null;
    }

}
```
## 滑动窗口的最大值
### 题目描述
给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。
### 分析

### 贴出代码
```java
import java.util.*;

public class Solution {
    public ArrayList<Integer> maxInWindows(int [] num, int size)
    {
        // 定义要返回的数组
        ArrayList<Integer> res = new ArrayList<>();
        // 如果size=0或者size > num的长度，则返回空的res
        if (size == 0 || size > num.length){
            return res;
        }

        for (int i = 0 ; i <= num.length - size ; i ++){
            // 自定义最大值为num[i]
            int max = num[i];
            // 滑动窗口，如果num[j] > max ，则交换max和num[j]
            for (int j = i + 1; j < i + size; j ++){
                if (num[j] > max){
                    max = num[j];
                }
            }
            // res添加max
            res.add(max);
        }

        return res;
    }
}
```

## 二维数组中的查找
### 题目描述
在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
### 分析
由题意可知，一行是递增的，所以一行一行的查找，采用二分查找思想。
### 贴出代码
```java
public class Solution {
    public boolean Find(int target, int [][] array) {
        for (int i = 0; i < array.length; i ++){
            int min = 0;
            int max = array[i].length - 1;
            while (min <= max){
                int mid = (min + max) / 2;
                if (target > array[i][mid]){
                    min = mid + 1;
                }else if (target < array[i][mid]){
                    max = mid - 1;
                }else
                    return true;
            }
        }
        return false;
    }
}
```
## 二叉搜索树的后序遍历序列
### 题目描述
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
### 分析
找住二叉查找树的特点：左子树<根<=右子树  使用分治思想。
### 贴出代码
```java
public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
        if (sequence.length == 0){
            return false;
        }
        if (sequence.length == 1){
            return true;
        }

        return judge(sequence,0,sequence.length - 1);
    }

    private boolean judge(int[] a, int start, int end){
        if (start >= end){
            return true;
        }
        int i = start;
        while (a[i] < a[end]){
            ++i;
        }
        for (int j = i; j < end; j++){
            if (a[j] < a[end]){
                return false;
            }
        }
        return judge(a,start,i-1) && judge(a, i, end - 1);
    }
}
```
## 按之字形顺序打印二叉树
### 题目描述
请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。
### 分析

### 贴出代码
```java
import java.util.*;
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        int layer = 1;

        Stack<TreeNode> s1 = new Stack<TreeNode>();
        s1.push(pRoot);

        Stack<TreeNode> s2 = new Stack<TreeNode>();

        ArrayList<ArrayList<Integer>> list = new ArrayList<ArrayList<Integer>>();

        while (!s1.empty() || !s2.empty()){
            if (layer % 2 != 0){
                ArrayList<Integer> temp = new ArrayList<>();
                while (!s1.empty()){
                    TreeNode node = s1.pop();
                    if (node != null){
                        temp.add(node.val);
                        System.out.println(node.val + " ");
                        s2.push(node.left);
                        s2.push(node.right);
                    }
                }
                if (!temp.isEmpty()){
                    list.add(temp);
                    layer++;
                    System.out.println();
                }
            }else{
                ArrayList<Integer> temp = new ArrayList<>();
                while (!s2.empty()){
                    TreeNode node = s2.pop();
                    if (node != null){
                        temp.add(node.val);
                        System.out.println(node.val + " ");
                        s1.push(node.right);
                        s1.push(node.left);
                    }
                }
                if (!temp.isEmpty()){
                    list.add(temp);
                    layer++;
                    System.out.println();
                }
            }
        }
        return list;
    }

}
```
## 树的子结构
### 题目描述
输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）
### 分析

### 贴出代码
```java
/**
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        boolean result = false;
        // 当Tree1 和 Tree2都不为零的时候，才进行比较，否则直接返回false
        if (root2 != null && root1 != null){
            // 如果找到了对应的Tree2的根节点的点
            if (root1.val == root2.val){
                // 以这个根节点为起点判断是否包含Tree2
                result = doesTree1HaveTree2(root1, root2);
            }
            // 如果找不到，那么就再去root的左儿子当作起点，去判断时候包含Tree2
            if (!result){
                result = HasSubtree(root1.left, root2);
            }
            // 如果还找不到，那么就再去root的右儿子当作起点，去判断时候包含Tree2
            if (!result){
                result = HasSubtree(root1.right,root2);
            }
        }
        return result;
    }

    public boolean doesTree1HaveTree2(TreeNode node1, TreeNode node2){
        // 如果Tree2已经遍历完了都能对应的上，返回true
        if (node2 == null){
            return true;
        }
        // 如果Tree2还没有遍历万，Tree1却遍历完了返回false
        if (node1 == null){
            return false;
        }
        // 如果其中有一个点没有对应上，返回false
        if (node1.val != node2.val){
            return false;
        }
        // 如果根结点对应的上，那么就分别去子节点里面匹配
        return doesTree1HaveTree2(node1.left, node2.left) && doesTree1HaveTree2(node1.right, node2.right);
    }
}
```
## 重建二叉树
### 题目描述
输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。
### 分析

### 贴出代码
```java
/**
 * Definition for binary tree
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
import java.util.*;
public class Solution {
    public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
        if (pre.length == 0 || in.length == 0){
             return null;
         }
         TreeNode node = new TreeNode(pre[0]);
         for (int i = 0; i < in.length ; i++){
             if (pre[0] == in[i]){
                 node.left = reConstructBinaryTree(Arrays.copyOfRange(pre,1,i+1),Arrays.copyOfRange(in,0,i));
                 node.right = reConstructBinaryTree(Arrays.copyOfRange(pre,i + 1,pre.length),Arrays.copyOfRange(in, i + 1,in.length));
             }
         }
         return node;
    }
}
```
## 机器人的运动范围
### 题目描述
地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？
### 分析
回溯
### 贴出代码
```java
public class Solution {
    public int movingCount(int threshold, int rows, int cols) {
        int flag[][] = new int[rows][cols];
        return helper(0, 0 , threshold, rows, cols, flag);
    }

    private int helper(int i, int j, int threshold, int rows, int cols, int[][] flag){
        if (i < 0 || i >= rows || j < 0 || j >= cols || numSum(i) + numSum(j) > threshold || flag[i][j] == 1) return 0;
        flag[i][j] = 1;
        return helper(i - 1, j, threshold, rows, cols, flag)
                + helper(i + 1, j, threshold, rows, cols, flag)
                + helper(i, j + 1, threshold, rows, cols, flag)
                + helper(i, j + 1, threshold, rows, cols, flag)
                + 1;
    }

    private int numSum(int i){
        int sum = 0;
        do{
            sum += i % 10;
        }while ((i = i /10) > 0);
        return sum;
    }
}
```
## 矩阵中的路径
### 题目描述
请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。 例如 a b c e s f c s a d e e 这样的3 X 4 矩阵中包含一条字符串"bcced"的路径，但是矩阵中不包含"abcb"路径，因为字符串的第一个字符b占据了矩阵中的第一行第二个格子之后，路径不能再次进入该格子。
### 分析
用一个状态数组保存之前访问过的字符，然后再分别按上，下，左，右递归。
### 贴出代码
```java
public class Solution {
    public boolean hasPath(char[] matrix, int rows, int cols, char[] str){
        int[] flag = new int[matrix.length];

        for (int i = 0; i < rows; i ++){
            for (int j = 0; j < cols; j ++){
                if (helper(matrix, rows, cols, i, j,str, 0, flag)){
                    return true;
                }
            }
        }
        return false;
    }

    private boolean helper(char[] matrix, int rows,int cols, int i, int j, char[] str, int k, int[] flag){
        int index = i * cols + j;

        if (i < 0 || i >= rows || j < 0 || j >= cols || matrix[index] != str[k] || flag[index] == 1){
            return false;
        }

        if (k == str.length - 1){
            return true;
        }

        flag[index] = 1;
        if (helper(matrix, rows, cols, i - 1, j, str, k + 1, flag)
                || helper(matrix, rows, cols, i + 1, j, str, k + 1, flag)
                || helper(matrix, rows, cols, i, j - 1, str, k + 1, flag)
                || helper(matrix, rows, cols, i, j + 1, str, k + 1, flag)){
            return true;
        }
        flag[index] = 0;

        return false;
    }
}
```
## 序列化二叉树
### 题目描述
请实现两个函数，分别用来序列化和反序列化二叉树
### 分析
根据前序遍历规则完成序列化与反序列化。所谓序列化指的是遍历二叉树为字符串；所谓反序列化指的是依据字符串重新构造成二叉树。依据前序遍历序列来序列化二叉树，因为前序遍历序列是从根结点开始的。当在遍历二叉树时碰到Null指针时，这些Null指针被序列化为一个特殊的字符“#”。另外，结点之间的数值用逗号隔开。
### 贴出代码
```java
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    int index = -1;

    String Serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        if (root == null){
            sb.append("#,");
            return sb.toString();
        }
        sb.append(root.val + ",");
        sb.append(Serialize(root.left));
        sb.append(Serialize(root.right));
        return sb.toString();
    }
    TreeNode Deserialize(String str) {
        index ++;
        String[] strr = str.split(",");
        TreeNode node = null;
        if (!strr[index].equals("#")){
            node = new TreeNode(Integer.valueOf(strr[index]));
            node.left = Deserialize(str);
            node.right = Deserialize(str);
        }
        return node;
    }
}
```
## 丑数
### 题目描述
把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。
### 分析
如果p是丑数，那么p=2^x * 3^y * 5^z
那么只要赋予x,y,z不同的值就能得到不同的丑数。
如果要顺序找出丑数，要知道下面几个特（fei）点（hua）。
对于任何丑数p：
（一）那么2*p,3*p,5*p都是丑数，并且2*p<3*p<5*p
（二）如果p<q, 那么2*p<2*q,3*p<3*q,5*p<5*q
现在说说算法思想：由于1是最小的丑数，那么从1开始，把2*1，3*1，5*1，进行比较，得出最小的就是1
的下一个丑数，也就是2*1，这个时候，多了一个丑数‘2’，也就又多了3个可以比较的丑数，2*2，3*2，5*2，
这个时候就把之前‘1’生成的丑数和‘2’生成的丑数加进来也就是
(3*1,5*1,2*2，3*2，5*2)进行比较，找出最小的。。。。如此循环下去就会发现，
每次选进来一个丑数，该丑数又会生成3个新的丑数进行比较。上面的暴力方法也应该能解决，但是如果在面试官用这种方法，估计面试官只会摇头吧。下面说一个O（n）的算法。在上面的特（fei）点（hua）中，既然有p<q, 那么2*p<2*q，那么
“我”在前面比你小的数都没被选上，你后面生成新的丑数一定比“我”大吧，那么你乘2生成的丑数一定比我乘2的大吧，那么在我选上之后你才有机会选上。其实每次我们只用比较3个数：用于乘2的最小的数、用于乘3的最小的数，用于乘5的最小的数就是比较(2*x , 3*y, 5*z) ，x>=y>=z的，重点说说下面代码中p的作用：int p[] = new int[] { 0, 0, 0 }; p[0]表示最小用于乘2比较数在数组a中的`位置`。
### 贴出代码
```java
public class Solution {
    final int d[] = {2, 3, 5};
    public int GetUglyNumber_Solution(int index) {
        if (index == 0){
            return 0;
        }
        int a[] = new int[index];
        a[0] = 1;
        int p[] = new int[] {0, 0, 0};
        int num[] = new int[] {2, 3, 5};
        int cur = 1;

        while (cur < index){
            int m = findMin(num[0], num[1],num[2]);
            if (a[cur - 1] < num[m]){
                a[cur ++] = num[m];
            }
            p[m] += 1;
            num[m] = a[p[m]] * d[m];
        }
        return a[index - 1];
    }

    private int findMin(int num2, int num3, int num5){
        int min = Math.min(num2, Math.min(num3,num5));
        return min == num2 ? 0 : min == num3 ? 1 : 2;
    }
}
```
## 链表中倒数第k个结点
### 题目描述
输入一个链表，输出该链表中倒数第k个结点。
### 分析

### 贴出代码
```java
/*
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}*/
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
        if(head == null || k <= 0){
            return null;
        }
        ListNode pre = head;
        ListNode last = head;
        for(int i = 1; i < k; i++){
            if(pre.next != null){
                pre = pre.next;
            }else{
                return null;
            }
        }
        while(pre.next != null){
            pre = pre.next;
            last = last.next;
        }
        return last;
    }
}
```
## 最小的K个数
### 题目描述
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。
### 分析

### 贴出代码
```java
import java.util.*;

public class Solution {
    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        if (k > input.length){
            return list;
        }
        Arrays.sort(input);
        for (int i = 0; i < k; i ++){
            list.add(input[i]);
        }
        return list;
    }
}
```
## 复杂链表的复制
### 题目描述
输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）
### 分析
评论区得来的答案
1. 在旧链表中创建新链表，此时不处理新链表的兄弟节点
2. 根据旧链表的兄弟节点，初始化新链表的兄弟节点
3. 从旧链表中拆分得来新链表
### 贴出代码
```java
/*
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
*/
public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
        if (pHead == null){
            return null;
        }
        RandomListNode currentNode = pHead;

        // 复制
        while (currentNode != null){
            RandomListNode cloneNode = new RandomListNode(currentNode.label);
            RandomListNode nextNode = currentNode.next;
            currentNode.next = cloneNode;
            cloneNode.next =nextNode;
            currentNode = nextNode;
        }

        currentNode = pHead;

        // 复制老节点的随机指针给新节点
        while (currentNode != null){
            currentNode.next.random = currentNode.random == null ? null : currentNode.random.next;
            currentNode = currentNode.next.next;
        }

        // 拆分链表，将链表拆分为原链表和复制后的链表
        currentNode = pHead;
        RandomListNode pCloneHead = pHead.next;
        while (currentNode != null){
            RandomListNode cloneNode = currentNode.next;
            currentNode.next = cloneNode.next;
            cloneNode.next = cloneNode.next == null ? null :cloneNode.next.next;
            currentNode = currentNode.next;
        }
        return pCloneHead;
    }
}
```
## 字符串的排列
### 题目描述
输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。
**输入描述：**
> 输入一个字符串,长度不超过9(可能有字符重复),字符只包括大小写字母。

### 分析

### 贴出代码
```java

```
## 正则表达式匹配
### 题目描述
请实现一个函数用来匹配包括'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配
### 分析

### 贴出代码
```java

```