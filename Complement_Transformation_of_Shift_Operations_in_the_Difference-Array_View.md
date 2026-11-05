---
title: 平移操作在差分视角下的补集转化
description: 差分意义下集合内元素的平移操作等价于补集元素的反操作
categories: [计算机算法]
tags: [Trick]
update_time: 2026-11-06
file_name: Complement_Transformation_of_Shift_Operations_in_the_Difference-Array_View
---

# 平移操作在差分视角下的补集转化

## 题意

[CF2278-D Lock and Unlock](https://codeforces.com/contest/2278/problem/D)

给出若干整数，要求通过一定操作后使得所有数变为相同值且花费最小。

每次操作方式：消耗 $k$ 点花费选择 $k$ 个数字保持不变，然后将所有未被锁住的数字同时 $+1$ 或 $-1$。

## Trick

题目仅要求操作后所有数字相同，而未要求操作后的最终数字，因此我们只需关注所有数字排序之后的差分值即可。

差分意义下，将 $n - 1$ 数字 $+1$ 或 $-1$，等价于将另外的 $1$ 个数字同时进行反操作。那么即可将题意等价转化为每次消耗 $1$ 点花费，将原数组的任意一个值 $+1$ 或 $-1$，最终让数组的所有数都相等。

