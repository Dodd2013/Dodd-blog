---
title: LeetCode 03 无重复字符的最长子串 - Dodd带你用JS刷LeetCode系列
tags: LeetCode
---

## 解题思路:

类似 kmp 和一些字符串算法，使用滑动窗口记录最大不重复的子串，枚举目标子串的头(i)尾(j)坐标，用一个 object 标记出现过的字符位置，当 j+1 没有出现在 object 中时 j++，反之设 i 为出现过字符的那个位置+1，记录过程中最大值，注意边界即可。

<!-- more -->

## 题目链接

[英文](https://leetcode.com/problems/longest-substring-without-repeating-characters/) [中文](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

## 题解

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
  let i = 0,
    j = 0,
    map = { [s[0]]: 0 },
    len = s.length,
    res = 0,
    index;
  while (i < len && j < len) {
    if (res < j - i + 1) {
      res = j - i + 1;
    }
    index = map[s[++j]];
    if (index >= i) {
      i = index + 1;
    }
    map[s[j]] = j;
  }
  return res;
};
```
