---
title: 字符串哈希
description: 字符串的哈希算法
category: 字符串
tags: [哈希, 字符串哈希]
update_time: 2026-08-28
file_path: String/StringHash
---

# 字符串哈希

**字符串哈希**（String Hash）是将一个任意长度字符串**映射**成一个在固定范围内的整数哈希值，以便在 $O(1)$ 时间内完成字符串比较的算法。

## 哈希函数

字符串哈希使用**多项式哈希取模**进行转换。

首先把字符集内的每个字符通过一定规则转换成 $base$ 范围内的唯一非零值 $v$，转换得到的值即为该字符在当前那一位的哈希值。

>   [!Warning]
>
>   转换后的值必须为非零值，因为当哈希值中出现前导零时，两个不同串的哈希值可能相同。

```cpp
// 当字符集只有小写字母时，base 范围一般为 [1, 26]
int turn(char c)
{
    return c - 'a' + 1;
}
```

对长度为 $n$ 的整个字符串进行哈希映射时，将字符串看作是一个 $n$ 位的 $base$ 进制整数，然后将该整数转换为十进制，并对模数 $m$ 取模，得到最终哈希值。
$$
\begin{aligned}
H(S) &= \sum_{i = 0}^{n - 1}(V[i] \cdot base^{n - i - 1}) \pmod{m}\\
     &= (V[0] \cdot base^{n - 1} + V[1] \cdot base^{n - 2} + \cdots + V[n - 1] \cdot base^0) \pmod{m}.\\
\end{aligned}
$$

```cpp
vector<int64_t> build(string s)
{
    int n = s.length();
    // prehashs[i] 表示从字符串开头到第 i 个位置结束这段位置的哈希值
    vector<int64_t> prehashs(n);
    for (int i = 0; i < n; ++i)
    {
        // base进制左移一位
        if (i)
            prehashs[i] = (prehashs[i - 1] * base) % mod;
        // 加上最低位
        prehashs[i] = (prehashs[i] + turn(s[i])) % mod;
    }
    return prehashs;
}
```

## 子串哈希

对于字符串 $S$ 的子串 $S[l,\ r]$，将其看作一个独立的字符串，根据哈希函数得到其哈希值表达式。
$$
\begin{aligned}
H(S[l,\ r]) &= \sum_{i = l}^{r - 1}(V[i] \cdot base^{r - i - 1}) \pmod{m}\\
            &= (V[l] \cdot base^{r - l - 1} + V[l + 1] \cdot base^{r - l - 2} + \cdots + V[r] \cdot base^0) \pmod{m}.\\
\end{aligned}
$$
将 $S[0,\ l - 1]$ 和 $S[0,\ r]$ 也看作一个独立的字符串，根据哈希函数得到各自表达式。
$$
\begin{aligned}
H(S[0,\ l - 1]) &= \sum_{i = 0}^{l - 2}(V[i] \cdot base^{l - i - 2}) \pmod{m}\\
                &= (V[l] \cdot base^{l - 2} + V[l + 1] \cdot base^{l - 3} + \cdots + V[l - 2] \cdot base^0) \pmod{m},\\

H(S[0,\ r]) &= \sum_{i = 0}^{r - 1}(V[i] \cdot base^{r - i - 1}) \pmod{m}\\
            &= (V[0] \cdot base^{r - 1} + V[1] \cdot base^{r - 2} + \cdots + V[r] \cdot base^0) \pmod{m}.\\
\end{aligned}
$$
两式相减得到，
$$
\begin{aligned}
H(S[0,\ r]) - H(S[0,\ l - 1]) &= (V[0] \cdot base^{r - 1} + V[1] \cdot base^{r - 2} + \cdots + V[l - 1] \cdot base^{r - l - 1}\\
                              &-\ V[0] \cdot base^{l - 2} - V[1] \cdot base^{l - 3} - \cdots - V[l - 1] \cdot base^0\\
                              &+\ V[l] \cdot base^{r - l - 1} + V[l + 1] \cdot base^{r - l - 2} + \cdots + V[r] \cdot base^0) \pmod{m}.\\
\end{aligned}
$$
发现对于 $[0,\ l - 1]$ 部分的 $base$， $H(S[0,\ r])$ 正好比 $H(S[0,\ l - 1])$ 每项多 $r - l + 1$ 次幂，由此可得，
$$
\begin{aligned}
H(S[0,\ r]) - base^{r - l + 1} \cdot H(S[0,\ l - 1]) &= (V[l] \cdot base^{r - l - 1} + V[l + 1] \cdot base^{r - l - 2} + \cdots + V[r] \cdot base^0) \pmod{m}.\\
\end{aligned}
$$
该表达式恰好与 $H(S[l,\ r])$ 相等。由此可得，
$$
H(S[l,\ r]) = (H(S[0,\ r]) - base^{r - l + 1} \cdot H(S[0,\ l - 1])) \pmod{m}.
$$
所以只需要求出字符串 $S$ 每个位置的**前缀哈希值**，就可以快速求出任意子串的哈希值。

```cpp
int64_t get_hash(int l, int r)
{
    int64_t prehash_l = l ? prehashs[l - 1] : 0;
    int64_t prehash_r = prehashs[r];
    // 用一个 powers 数组存储在模 mod 意义下 base 的若干幂次的值
    return ((prehash_r - (prehash_l * powers[r - l + 1]) % mod) % mod + mod) % mod;
}
```

>   [!Tip]
>
>   另一种哈希的方式是将 $base$ 的幂次从左到右依次从 $0$ 开始增加，这样幂次与子串的相对下标相等，逻辑更清晰。
>
>   但并不建议这种写法，因为求子串哈希值时需要将 $Hash(S[0, r])$ 除以 $r - l + 1$。同余除法运算需要额外求出 $base$ 的幂次的逆元。

