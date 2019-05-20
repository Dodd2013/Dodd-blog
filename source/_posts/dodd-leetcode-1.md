---
title: LeetCode 01 两数之和 - Dodd带你用JS刷LeetCode系列
tags: LeetCode
---

## 解题思路:

这个题给出一个数组和一个数 sum，要求从数组中找出两个数之和为 sum，遍历一遍数组，同时维护出现过的数字的 map(value 为其 index)和当前数字需要互补的数字即可。

<!-- more -->
## 题目链接

[英文](https://leetcode.com/problems/two-sum/) [中文](https://leetcode-cn.com/problems/two-sum/)

## 题解

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
const twoSum = function(nums, target) {
  const map = {};
  let res;
  nums.forEach((num, index) => {
    if (map[num] !== undefined) {
      res = [map[num], index];
      return;
    } else {
      map[target - num] = index;
    }
  });
  return res;
};
```
