# 35. [搜索插入位置](https://leetcode.cn/problems/search-insert-position/description/?envType=study-plan-v2&envId=top-100-liked)

## 题目描述
* 给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
* 请必须使用时间复杂度为 $O(\log n)$ 的算法。
 
### 示例
* **示例 1:**
    * 输入: nums = [1,3,5,6], target = 5
    * 输出: 2
* **示例 2:**
    * 输入: nums = [1,3,5,6], target = 2
    * 输出: 1
* **示例 3:**
    * 输入: nums = [1,3,5,6], target = 7
    * 输出: 4
 
### 限制
* $1 \le \text{nums.length} \le 10^4$
* $-10^4 \le \text{nums}[i] \le 10^4$
* `nums` 为 **无重复元素** 的 **升序** 排列数组
* $-10^4 \le \text{target} \le 10^4$

## 解法思路
题目明确要求算法的时间复杂度必须是 $O(\log n)$，同时数组是有序的，这两个条件强烈暗示本题必须使用**二分查找**算法。

本题的本质是：**寻找数组中第一个大于或等于 `target` 的元素的下标**（也就是 C++ 中的 `lower_bound` 语义）。如果数组中所有元素都小于 `target`，则说明目标值应该被插入到数组的最后，此时返回数组的长度 `len(nums)`。

**为什么不在二分的过程中，找到 `target` 就立刻返回？**
在无重复元素的数组中，立刻返回是可以的。但如果为了代码的普适性（例如数组中有重复元素时），不提前返回，而是严格压缩区间直到左右指针相遇，可以保证我们找到的永远是**第一个**等于 `target` 的元素的下标。这种写法在解决更复杂的二分查找问题时非常有用。

这道题的二分查找有三种常见的区间定义写法：**闭区间**、**左闭右开区间**和**开区间**。这里主推最符合人类直觉的**闭区间 `[left, right]`** 写法，同时补充了 Python 极其方便的内置库函数写法。

### 算法步骤 (以闭区间为例)
1. 初始化左边界 `left = 0`，右边界 `right = len(nums) - 1`，定义在闭区间 `[left, right]` 中搜索。
2. 只要区间不为空（`while left <= right`），就继续循环：
    * 计算中间位置 `mid = (left + right) // 2`。
    * 如果 `nums[mid] < target`：说明目标值应该在 `mid` 的右侧。由于 `mid` 已经不是答案，我们将搜索范围缩小到 `[mid + 1, right]`，即令 `left = mid + 1`。
    * 如果 `nums[mid] >= target`：说明目标值就是 `mid` 或者在 `mid` 的左侧。此时 `mid` 可能是我们要找的位置，我们保留它及它左边的部分，搜索范围缩小到 `[left, mid - 1]`，即令 `right = mid - 1`。
3. 当循环结束时，`left` 指针刚好指向第一个大于等于 `target` 的位置，返回 `left` 即可。

### 复杂度分析
* **时间复杂度:** **O(log N)**，其中 N 为数组 `nums` 的长度。每次循环都将搜索区间缩小一半，这是标准的二分查找时间复杂度。
* **空间复杂度:** **O(1)**，除了几个用于记录区间边界的指针变量外，没有使用额外的空间。

## 代码实现

### python
```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        """
        解法一：标准二分查找（闭区间写法 [left, right]）
        寻找第一个大于等于 target 的元素的下标
        """
        left = 0
        right = len(nums) - 1
        
        while left <= right:
            mid = (left + right) // 2
            if nums[mid] < target:
                # mid及之前的值都小于target，更新下界
                left = mid + 1
            else:
                # mid处的值 >= target，可能是插入位置，继续向左逼近
                right = mid - 1
                
        # 循环结束时，left 指向的一定是第一个 >= target 的位置
        return left

# ==========================================
# 补充：Python 库函数写法 (一行代码秒杀)
# ==========================================
# import bisect
#
# class Solution:
#     def searchInsert(self, nums: List[int], target: int) -> int:
#         # bisect_left 的底层实现逻辑与上面的二分查找完全一致
#         return bisect.bisect_left(nums, target)
```