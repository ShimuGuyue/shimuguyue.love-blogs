---
title: 分段筛法
description: 分段二次筛取较大范围内小区间质数筛法
category: 数论
tags: [二次筛法, 分段筛法, 欧拉筛, 筛法, 质数, 质数筛]
update_time: 2026-09-02
file_path: NumberTheory/SegmentedSieve
---

# 分段筛法

对于一般的质数筛法，如埃式筛或欧拉筛，他们每秒只能处理约 $10^7$ 数量级范围内的质数筛取。

当待筛取质数的数量级较大时，尽管这些筛法难以奏效，但若筛选的区间长度较小，仍可通过一些技巧让其发挥作用。

当要筛取一个区间 $[L,\ R]$ 内的所有质数，条件满足 $R - L \leq 10 ^ 7$ 时，可通过先筛出 $[1,\ \sqrt R]$ 内的所有质数，再将这些质数在对应区间内的倍数标记为非质数，即第二次筛取。

这种分段筛取的方式即为**分段筛法**，因其共进行两次筛取，也可称为**二次筛法**。

## 初筛

首先通过一般筛法筛取出 $[1,\ \sqrt R]$ 内的所有质数，此处不详细展开。

```cpp
vector<int64_t> primes1 = seiving(sqrt(R));
```

## 二筛

将初筛的质数集合中的质数在区间内的倍数标记为非质数。本质上是简化版的**埃式筛**。

```cpp
vector<bool> is_prime(R - L + 1, true); // 对待筛区间建立映射
vector<int64_t> primes2;

void seiving2()
{
    for (int64_t p : primes1)
    {
        // 找到区间内最小的 p 的倍数
        int64_t begin = (L - 1) / p * p + p;
        for (int64_t i = begin; i <= R; i += p)
        {
            is_primes[i - L] = false;
        }
    }
    // 记录所有质数
    for (int64_t i = L; i <= R; ++i)
    {
        if (is_prime[i - L])
            primes2.push_back(i);
    }
}
```

分段筛法的时间复杂度取决于初筛使用的筛法和区间长度。设初筛使用的筛法时间复杂度为 $O(P(n))$，区间长度为 $N$，则分段筛法的总时间复杂度为 $O(P(\sqrt R) + N \log \log R)$

