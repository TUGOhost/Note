LeetCode

## 最大子序和

### 题目描述

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:
```
输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```
### 分析
最大子序列是要找出由数组成的一维数组中和最大的连续子序列。比如{5,-3,4,2}的最大子序列就是 {5,-3,4,2}，它的和是8,达到最大；而 {5,-6,4,2}的最大子序列是{4,2}，它的和是6。你已经看出来了，找最大子序列的方法很简单，只要前i项的和还没有小于0那么子序列就一直向后扩展，否则丢弃之前的子序列开始新的子序列，同时我们要记下各个子序列的和，最后找到和最大的子序列。
动态方程： 
最大子序列和是连续的子序列 
ThisSum[i]表示第i处，以A[i]结尾的子序列的最大和。 
则状态方程为ThisSum[i]=max(ThisSum[i-1]+nums[i],nums[i])
也即是说，如果ThisSum[i-1]<0则ThisSum[i]就是nums[i],否则ThisSum[i]=nums[i]+ThisSum[i-1]（因为是以i结尾的
### 代码
```java
package leetcode;

public class maxSubArray {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[] nums = {-2,1,-3,4,-1,2,1,-5,4};
		System.out.println(maxSubArray(nums));
	}
	public static int maxSubArray(int[] nums) {
        int r,l;
        int maxSum = nums[0],thisSum = nums[0];
        int new_l = 0;
        for(int i = 1; i < nums.length; i++) {
        	if(thisSum < 0) {
        		thisSum = nums[i];
        		new_l = i;
        	}else {
        		thisSum += nums[i];
        	}
        	if(thisSum > maxSum) {
        		maxSum = thisSum;
        		l = new_l;
        		r = i;
        	}
        }
        
        return maxSum;
    }
}

```
## 加一

### 题目描述

给定一个**非负整数**组成的**非空**数组，在该数的基础上加一，返回一个新的数组。

最高位数字存放在数组的首位， 数组中每个元素只存储一个数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

**示例 1:** 

```
输入: [1,2,3]
输出: [1,2,4]
解释: 输入数组表示数字 123。
```

**示例 2:** 

```
输入: [4,3,2,1]
输出: [4,3,2,2]
解释: 输入数组表示数字 4321。
```

### 分析

这题，一开始以为是给最后一个数+1，没想到是进位的问题，题目理解真是一知半解。。。语文还是要好好学的啊。

下面这个是我在百度中了解的：

将一个数字的每个位上的数字分别存到一个一维向量中，最高位在最开头，我们需要给这个数字加一，即在末尾数字加一，如果末尾数字是9，那么则会有进位问题，而如果前面位上的数字仍为9，则需要继续向前进位。

具体算法如下：首先判断最后一位是否为9，若不是，直接加一返回，若是，则该位赋0，再继续查前一位，同样的方法，知道查完第一位。如果第一位原本为9，加一后会产生新的一位，那么最后要做的是，查运算完的第一位是否为0，如果是，则在最前头加一个1。 

### 贴出来代码

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int n = digits.length;
		for(int i = digits.length -1; i>=0;--i) {
			if(digits[i]<9) {
				++digits[i];
				return digits;
			}
			digits[i] =0;
		}
		int[] res = new int[n+1];
		res[0] =1;
		return res;
    }
}
```

## 合并两个有序数组

### 题目描述

给定两个有序整数数组 *nums1* 和 *nums2*，将 *nums2* 合并到 *nums1* 中*，*使得 *num1* 成为一个有序数组。

**说明:**

- 初始化 *nums1* 和 *nums2* 的元素数量分别为 *m* 和 *n*。
- 你可以假设 *nums1* 有足够的空间（空间大小大于或等于 *m + n*）来保存 *nums2* 中的元素。

**示例:**

```
输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

输出: [1,2,2,3,5,6]
```

### 分析

***思路1：***

```
先合并两个数组，再来排序
```

***思路2：***

```
从后往前填充即可

先当两个数组都有元素的时候填充大的到末尾，如果有一个数组的数用完了，说明剩下的那个数组的所有数都小于当前填充的位置：
如果是第一个数组用完了，说明剩下的(最小的那些)全是数组2，把数组2填充进去就好了
如果是第二个数组用完了，说明剩下的全是数组1，不用填充了，他们已经在了
```

### 贴出来代码

```java
public class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1, j = n - 1, writeIdx = m + n - 1;
        while (i >= 0 && j >= 0)
            nums1[writeIdx--] = nums1[i] > nums2[j]? nums1[i--] : nums2[j--];
        while (j >= 0)
            nums1[writeIdx--] = nums2[j--];
    }
}
```

