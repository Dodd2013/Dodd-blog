---
title: LeetCode 02 两数相加 - Dodd带你用JS刷LeetCode系列
date: 2019-5-20
tags: LeetCode
---

## 解题思路:

链表型大数模拟相加，记住不能直接转换数字相加后再转换回来，应该会溢出(我没有试~逃.jpg), 通过率比较低的原因可能还有就是忘记了最后一位的进位

<!-- more -->

## 题目链接

[英文](https://leetcode.com/problems/add-two-numbers/) [中文](https://leetcode-cn.com/problems/add-two-numbers/)

## 题解

```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */

/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function(l1, l2) {
  let head = new ListNode();
  let overflow = 0;
  let cur = head;
  // Don't forget overflow here!!!!!
  while (l1 || l2 || overflow) {
    cur.next = new ListNode(0);
    cur = cur.next;
    if (l1) {
      cur.val += l1.val;
      l1 = l1.next;
    }
    if (l2) {
      cur.val += l2.val;
      l2 = l2.next;
    }
    cur.val += overflow;
    if (cur.val > 9) {
      cur.val -= 10;
      overflow = 1;
    } else {
      overflow = 0;
    }
  }
  return head.next;
};
// test case
// function ListNode(val) {
//   this.val = val;
//   this.next = null;
// }
// let l1 = new ListNode(5);
// l1.next = new ListNode(5);
// l1.next.next = new ListNode(5);
// let l2 = new ListNode(5);
// l2.next = new ListNode(5);
// l2.next.next = new ListNode(5);

// let l3 = addTwoNumbers(l1, l2);
// while (l3) {
//   console.log(l3.val + "->");
//   l3 = l3.next;
// }
```
