



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
//评论中很简短的一个写法
class Solution {
public:
    int TreeDepth(TreeNode* pRoot){
        if(!pRoot) return 0 ;
            return max(1+TreeDepth(pRoot->left), 1+TreeDepth(pRoot->right));
    }
};
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

