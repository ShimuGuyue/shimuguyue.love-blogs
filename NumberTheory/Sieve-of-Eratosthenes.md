---
title: 埃式筛筛质数
description: 小范围质数筛法
category: 数论
tags: [埃式筛, 筛法, 质数, 质数筛]
update_time: 2026-08-28
file_path: NumberTheory/Sieve-of-Eratosthenes
---

# 埃式筛

**埃式筛**（Sieve of Eratosthenes）是一种高效筛选出 $10 ^ 7$ 内所有素数的算法。

由于质数的倍数（不含本身）一定是合数，所以从 $2$ 开始遍历所有数，如果遇到一个质数，将他的所有倍数标记为合数。

如果有一个数在遍历过程中没有被标记为合数，说明比他小的数里面没有他的因数，这个数是质数。

根据梅滕斯第二定理，该算法的时间复杂度为 $O(n \log \log n)$，稍逊于线性筛的 $O(n)$，但在 $10 ^ 7$ 数据范围内差距很小，且代码量更简短。

**优化一**：对于任意 $a \times b = c,\ a > b$，在 $c$ 被 $a$ 标记之前一定已经被 $b$ 标记过，因此遍历 $a$ 的倍数时从 $a ^ 2$ 开始即可。

**优化二**：除 $2$ 以外的所有偶数均为合数，除 $2$ 以外的质数只会出现在奇数中，因此只需要遍历每个质数的奇数倍即可。

```cpp
constexpr int SIZE = 10000000; // 筛质数的范围

vector<int> primes;     // 存储所有质数
vector<bool> is_primes; // 判断是否是质数

void sieving()
{
    is_primes.assign(SIZE + 1, true);

    // 特判唯一的偶数和 1
    is_primes[1] = false;
    primes.push_back(2);
    for (int i = 3; i <= SIZE; ++i)
    {
        if (i % 2 == 0) // 偶数，跳过
        {
            is_primes[i] = false;
            continue;
        }
        if (!is_primes[i]) // 被判定为合数，跳过
            continue;
        // 记录质数
        primes.push_back(i);
        // 将质数的倍数标记为合数
        if (int64_t(i) * i > SIZE)
            continue;
        for (int j = i * i; j <= SIZE; j += i * 2) // 只需标记奇数
        {
            is_primes[j] = false;
        }
    }
}
```
