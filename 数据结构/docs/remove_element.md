# 移除元素

> 题目

```txt
1. 给定一个数组 nums 和一个值 val，你需要原地移除所有数值等于 val 的元素，返回移除后数组的新长度。

2. 不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

3. 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。
```

> 审题

```txt
1. 不能创建新的数组空间
2. O(1)条件下
3. 元素的顺序可以改变
```

> 思路

```txt
双指针

```

> 实现

```python
class Solution(object):
    def removeElement(self, nums, val):
        """
        :type nums: List[int]
        :type val: int
        :rtype: int
        """
        j = 0
        for i in range(len(nums)):
            if nums[i] != val:
                nums[j] = nums[i]
                j+=1
        return j

nums = [0,1,2,2,3,0,4,2]
val = 2

print(Solution().removeElement(nums, val))
```
