---
title: KMP
description: 根据模式串前缀信息在文本串中快速检索
category: 字符串
tags: [字符串匹配, KMP, 前缀]
update_time: 2026-08-29
file_path: String/KMP
---

# KMP

**KMP**（Knuth-Morris-Pratt）算法是一种根据模式串的**前缀信息**，在文本串中快速检索对应模式串的算法。

## border

如果字符串 $S$ 的一个子串**既是它的前缀，又是它的后缀**，则该子串称为 $S$ 的一个 **border**。

* 一般地，**字符串本身**不算是他的 border，具体根据语境判断。
* 特殊地，border 也可以指这个字符串串的**长度**，具体根据语境判断。
* 一般地，border 通常指字符串地最大 border，具体根据语境判断。

>   $S$：
>
>   `b b a b b a b`
>
>   $S$ 的所有 border：
>
>   `b b a b b a b`
>   `b b a b`
>   `b`

>   [!NOTE]
>
>   **border 的传递性**：$S$ 的 border 的 border 也是 $S$ 的 border。

对于 $S$ 的任意前缀串 $S[0,\ i]$，如果 $S[0,\ i - 1]$ 的任意一个 border 的下一个字符与 $S_i$ 相等，则该 border 加上下一个字符是 $S[0,\ i]$ 的一个border。

>   $S[0, i]$：`a a b a a b`
>
>   $S[0,\ i - 1]$ 的其中最大 border 是 `a a`，下一个字符 `b` 与 $S_i$ 相等，加上该字符形成前缀串 `a a b`，也是 $S[0,\ i]$ 的一个后缀串，即 $S[0,\ i]$ 的一个 border。

```cpp
vector<int> get_borders(string& s)
{
    int n = s.length();
    vector<int> borders(n);
    for (int i = 1; i < n; ++i)
    {
        int border = borders[i - 1];
        while (border != 0 && s[i] != s[border])
        {
            border = borders[border - 1];
        }
        if (s[i] == s[border])
            ++border;
        borders[i] = border;
    }
    return borders;
}
```

根据 border 的传递性，模式串上的每个节点都可以通过若干条最大 border 链连接在一起形成一个树形结构，称为 **border 树**。KMP 的匹配过程就建立在 border 树的结构基础上。

## KMP匹配

KMP 算法中，使用两个指针分别表示主串和模式串当前遍历到的位置。与暴力匹配不同，KMP 算法中主串指针不断前进，仅有模式串的指针在字符匹配失败时进行回溯。

当在模式串的 $i$ 位置匹配失败时，前 $i - 1$ 个位置是匹配成功的。在 $i - 1$ 位置处，该位置后缀 border 部分可以作为新一次匹配中已经匹配好的前缀部分继续匹配。模式串指针 $i$ 根据 borders 数组跳转到下一个匹配位置 `borders[i - 1]`。

若下一个位置匹配成功，主串和模式串指针均后移一位，否则继续跳转到下一个匹配位置。除非所有的位置与文本串当前字符都不匹配，主串指针后移一位，模式串指针回到起点。

>   主串：
>
>   `a a b a a b a a f`
>
>   模式串：
>
>   `a a b a a f`
>
>   模式串的 borders 数组：
>
>   `0 1 0 1 2 0`
>
>   当主串指针和模式串指针分别为 $i = 5,\ j = 5$ 时，模式串已完成匹配 `aabaa`，下一个字符 `f` 匹配失败。
>
>   由于串 `aabaa` 的最大 border 为 $2$，因此模式串指针 $j$ 跳转到第三个位置尝试继续匹配。

>   [!Tip]
>
>   由于 borders 数组指示了模式串指针匹配失败时跳转的位置，因此也称为 next 数组或 fail 数组。

```cpp
string pattern;
vector<int> nexts = get_borders(pattern);

int kmp(string& text)
{
    int n = text.length();
    int m = nexts.length();

    int i = 0, j = 0;
    while (i < n)
    {
        if (text[i] == pattern[j])
        {
            // 当前位置匹配成功，尝试匹配下个位置
            ++i;
            ++j;
            // 模式串匹配完成，返回主串中匹配到的起始位置。
            // 如果要找到所有匹配的位置，则存储每个找到的位置，
            // 然后类比匹配失败的情况跳转模式串指针，继续下一轮匹配
            if (j == m)
                return i - m;
        }
        else
        {
            // 匹配不成功则跳转 j 指针
            if (j != 0)
                j = nexts[j - 1];
            // 如果第一个字符都不匹配，则仅 i 指针后移
            else
                ++i;
        }
    }
    // 匹配失败返回 -1
    return -1;
}
```

## 自动机优化

由于 KMP 算法在匹配失败时需要沿 border 链多次跳转，在极端数据情况下匹配失败时，模式串指针可能会连续回退多次导致匹配单个字符时的时间发生波动，因此可以根据 border 树上的节点进行预处理，找出当已经匹配成功任意数量个字符时，再匹配下一个字符时，对于可能出现的任意字符，模式串指针应该跳转到什么位置，以便在匹配时一步到位，无需多次跳转。

预处理时，需要一个一维数组 fails 来存储每个节点的最大 border，以及一个二维数组 nexts 来表示每个位置 $i$ 匹配到每个字符 $c$ 时应跳转的位置 `nexts[i][c]`。

**逐字符**对 nexts 数组进行预处理时，当匹配到第 $i$ 个字符时，若匹配新字符 $ch$ 成功时，设置 `nexts[i][ch] = i + 1`，成功匹配的字符数计数加一，模式串匹配的指针后移；否则，$i$ 随 $i - 1$ 处的 fail 链跳转。由于 `fail[i]` 处的信息已被处理过且包含所有更小 border 处的信息，所以 fail 链只需一次跳转即可得到 `nexts[i][ch]` 指向的位置，即 `nexts[i][ch] = nexts[fail[i - 1]][ch]`。

对于模式串的匹配指针 $i$，正在匹配第 $i$ 个位置同样也可以表示已经匹配了 $i$ 个字符。而对于一个长度为 $n$ 的模式串，统计计数共计有 $n + 1$ 种不同情况。当需要匹配所有出现的字符串时，应将 nexts 数组的第一维长度设为 $n + 1$，用 `nexts[n][ch]` 表示已经匹配成功一个模式串之后再匹配下一个模式串应该跳转到哪个位置，以便匹配所有模式串的位置。（或者不额外开一个位置，匹配完成时根据 fail 链回退亦可。）

将 KMP 优化为有限状态自动机后，单次匹配的时间复杂度从均摊 $O(1)$ 优化为严格 $O(1)$，额外空间复杂度为 $O(n\ base)$，其中 $base$ 为字符集大小。

```cpp
vector<array<int, 26>> build_next(string& pattern, vector<int>& fails)
{
    int m = pattern.length();

    vector<array<int, 26>> nexts(m + 1);
    for (int i = 0; i <= m; ++i)
    {
        for (int index = 0; index < 26; ++index)
        {
            if (index + 'a' == pattern[i])
                if (i == m) //  完全匹配成功尝试下一次匹配
                    nexts[i][index] = nexts[fails[i - 1]][index];
                else
                    nexts[i][index] = i + 1;
            else
                if (i == 0)
                    nexts[i][index] = 0;
                else
                    nexts[i][index] = nexts[fails[i - 1]][index];
        }
    }
    return nexts;
}
```

匹配过程不同于朴素 KMP 的指针跳转，文本串指针向前走的同时，模式串指针根据预处理好的 nexts 数组一步跳转到位。

```cpp
vector<int> kmp(string& text)
{
    int n = text.length();
    int m = pattern.length();

    vector<int> indexs;
    int index = 0;
    for (int i = 0; i < n; ++i)
    {
        index = nexts[index][text[i] - 'a'];
        if (index == m)    // 若只找第一个出现位置则直接返回
             indexs.push_back(i - m + 1);
    }
    return indexs;
}
```
