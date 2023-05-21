---
title: LeetCode题解 - 01.两数之和
categories: LeetCode
tags:
  - LeetCode
  - 算法
description: ''
toc: true
date: 2020-11-27 12:05:13
---

# 题目
[1.两数之和](https://leetcode-cn.com/problems/two-sum)
难度：`简单`

给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 `两个` 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

 
```text
示例:

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

# 题解

## 暴力法

> 确定第一个元素`i`，然后搜索其后的元素`j`，当满足条件`nums[i] + nums[j] == target`时，返回`[i,j]`

- 时间复杂度：O(n^2^)
- 空间复杂度：O(1)

```python3
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        length = len(nums)
        for i in range(length):
            for j in range(i + 1, length):
                if nums[i] + nums[j] == target:
                    return [i, j]
```

## 哈希表

> 将元素放入`HashMap`中，使用元素值作为Key，元素的索引作为Value
> 查找时当使用`target - num[i]`为Key的元素存在则返回`[HashMap[target - num[i]],num[i]]`

- 时间复杂度 O(n)
- 空间复杂度 O(n)

> 使用测试样例测试返回值为`[1,0]`,与给定结果的`[0,1]`顺序不符，但也能通过。

```python3
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        table = dict()
        for i in range(len(nums)):
            n = nums[i]
            tmp = target - n
            if tmp in table:
                return [i, table[tmp]]
            else:
                table[n] = i
```