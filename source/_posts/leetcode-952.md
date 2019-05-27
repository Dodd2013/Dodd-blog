---
title: LeetCode 952. 按公因数计算最大组件大小 - Dodd带你用JS刷LeetCode系列
tags: LeetCode
---

## 解题思路:

这个题用最大公约数去循环暴力并查集会超时，所以可以先用素数筛筛出范围以内所有素数(leetcode的test case有bug，看代码注释),利用素数来做并查集，合并的同时记录数的个数count，注意合并的时候个数也要合并。

<!-- more -->

## 题目链接

[英文](https://leetcode.com/problems/largest-component-size-by-common-factor/) [中文](https://leetcode-cn.com/problems/largest-component-size-by-common-factor/)

## 题解

```javascript
/**
 * @param {number[]} A
 * @return {number}
 */
var largestComponentSize = function(A) {
  let flag = [];
  let group = [];
  let su = [false, false];
  let suArray = [];
  let maxNum = 100000;
  let count = {};
  // 素数筛法，筛出素数数组suArray
  for (let i = 2; i <= maxNum; i++) {
    if (su[i] === undefined) {
      su[i] = true;
      suArray.push(i);
      for (let j = 2; j * i <= maxNum; j++) {
        su[j * i] = false;
      }
    }
  }
  // delete su; 这里要是环境GC(垃圾回收)做的不好的话可以手动删一下这个回收内存

  // 并查集find
  function find(root) {
    let son = root;
    let t;
    while (root != flag[root]) {
      root = flag[root];
    }
    while (son != root) {
      tmp = flag[son];
      flag[son] = root;
      son = tmp;
    }
    return root;
  }
  // 并查集union
  function union(root1, root2) {
    let x, y, t;
    x = find(root1);
    y = find(root2);
    if (y > x) {
      t = y;
      y = x;
      x = t;
    }
    if (x != y) {
      flag[x] = y;
      if (count[y]) {
        if (count[x]) {
          count[y] += count[x];
          delete count[x];
        }
      } else {
        count[y] = 1;
      }
    }
  }

  // 初始化并查集
  for (let i = 0; i < suArray.length; i++) {
    flag[i] = i;
  }
  // 循环每个数字
  for (let i = 0; i < A.length; i++) {
    let pre = -1; // 记录上次整除的素数在suArray中的位置
    // 遍历素数数组suArray,注意多加一个条件可以加速
    for (let j = 0; j < suArray.length && A[i] > 1; j++) {
      if (A[i] % suArray[j] === 0) {
        // 如果这个数字之前已经有整除的素数，那么合并当前找到的素数和之前的素数所在的集合
        if (pre !== -1) {
          union(pre, j);
        }
        // 如果这个是第一个找到的因子，那么就让计数器加一
        if (pre === -1) {
          if (count[find(j)]) {
            count[find(j)]++;
          } else {
            count[find(j)] = 1;
          }
        }
        pre = j;
        // 循环除去当前素数，加快速度
        while (A[i] % suArray[j] === 0) A[i] /= suArray[j];
      }
    }
  }
  // 找到计数器中最大值就是答案
  let max = 0;
  for (let key in count) {
    if (count[key] > max) max = count[key];
  }
  return max;
};

// test case
console.log(largestComponentSize([20, 50, 9, 63]));

```
