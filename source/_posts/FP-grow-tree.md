---
title: 数据挖掘 关联规则的FP-growth-tree（FP增长树）的python实现 使用方法
date: 2016-3-31
tags: 数据挖掘
---

## 代码地址

> [点我去找代码](http://git.oschina.net/Dodd/FP-grow-tree)

<!--more-->

## 目录结构

1.  FP_Grow_tree.py :入口类
2.  fptree.py：构建树的类
3.  node.py 节点数据结构类
4.  sample.py：样例类
5.  tree.py：FP-grow 算法类生成模式基数据集
6.  unit.py：组合函数和子集函数

## 调用方法

导入 FP_Grow_tree.py 类，下边代码为 sample.py 的实例

```python
#-*- coding:utf-8 –*-

import FP_Grow_tree
sample=[
	['milk','eggs','bread','chips'],
	['eggs','popcorn','chips','beer'],
	['eggs','bread','chips'],
	['milk','eggs','bread','popcorn','chips','beer'],
	['milk','bread','beer'],
	['eggs','bread','beer'],
	['milk','bread','chips'],
	['milk','eggs','bread','butter','chips'],
	['milk','eggs','butter','chips']
]
sample1=[
	[u'牛奶',u'鸡蛋',u'面包',u'薯片'],
	[u'鸡蛋',u'爆米花',u'薯片',u'啤酒'],
	[u'鸡蛋',u'面包',u'薯片'],
	[u'牛奶',u'鸡蛋',u'面包',u'爆米花',u'薯片',u'啤酒'],
	[u'牛奶',u'面包',u'啤酒'],
	[u'鸡蛋',u'面包',u'啤酒'],
	[u'牛奶',u'面包',u'薯片'],
	[u'牛奶',u'鸡蛋',u'面包',u'黄油',u'薯片'],
	[u'牛奶',u'鸡蛋',u'黄油',u'薯片']
]
#print(sample1)
##参数说明 sample为事务数据集 []为递归过程中的基,support为最小支持度
support=3
ff=FP_Grow_tree.FP_Grow_tree(sample1,[],support)
##打印频繁集
ff.printfrequent()
```

## 依赖环境

1.  python3.x
2.  python itertools
3.  python operator
    装完 python3.x 好像第二第三都会自带。

## 总结

有空实现实现书本上的算法也是极好的，可以锻炼一下能力，渣渣写的很乱，大神看了不要见怪。
