



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

## 输入描述:

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

