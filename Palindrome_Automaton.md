---
title: 回文自动机
description: 以字典的形式记录字符串中所有回文子串的回文信息。
categories: [数据结构]
tags: [字符串, 回文, 树, 自动机]
update_time: 2026-10-19
file_name: Palindrome_Automaton
---

# 回文自动机

**回文自动机**（Palindrome Automaton, PAM）是一种用于记录字符串中所有**本质不同回文子串**的树形数据结构，维护包括出现次数、长度等信息。

由于回文串左右对称，当回文串进行拓展时，左半串在首部添加一个字符，右半串在尾部添加一个字符，因此可用回文串的右半部分来表示整个回文串。

但由于回文串的长度根据是奇数还是偶数不同，最中间字符的出现次数不同，需要作出区分。回文自动机对此做出的优化是将自动机建立为两棵树，两个根节点分别称为**奇根**和**偶根**。其中偶根的长度为 $0$；奇根的长度为 $-1$，是一个虚根，由其拓展出的回文串长度为奇数。

>   另一种做法是像 Manacher 算法那样将字符串进行预处理，这样原来所有的回文串都可以表示为奇回文串。但这种转换过于繁琐，不推荐。

与 Trie 类似，PAM 上的每个节点表示一个回文串。用 $\operatorname{S}(u)$ 表示节点 $u$ 代表的回文串，从根节点到节点 $u$ 的所有字符按顺序连接成的字符串表示回文串 $\operatorname{S}(u)$ 的右半部分，若回文串长度为奇数，则包括中间的那一个字符。

PAM 上每个节点通过若干条**后继边**指向新节点，新节点代表的回文串等价于在父节点的回文串首尾各添加一个后继边上的字符。若节点 $u$ 通过字符 $ch$ 表示的边指向节点 $v$，则 $\operatorname{S}(v) = \textbf{ch} + \operatorname{S}(u) + \textbf{ch}$，$\operatorname{len}(v) = \operatorname{len}(u) + 2$。其中，奇根的直接后继节点表示的回文串长度为 $1$，即单独的一个字符构成的回文串。

>   [!Note]
>
>   奇根的长度为 $-1$，可理解为由奇根拓展出的回文串删去了一个中心字符，便将一个偶回文串变成了一个奇回文串。

PAM 上的每个节点都有一个**失配指针**，指向在当前节点所表示的回文串向两侧拓展失败时跳转的位置。对于节点 $u$ 的失配边指向的节点 $v$，$\operatorname{S}(v)$ 应是 $\operatorname{S}(u)$ 的**非平凡最大回文后缀**，同时也是**非平凡最大 border**。

## 构建 PAM

PAM 上的每个节点需要维护的基础信息为回文串长度、失配指针和后继边。

```cpp
struct Data
{
    int len   = 0;
    int fail  = 0;
    // nexts 初始化全 0，因不会有节点拓展出空串，
    // 当后继边指向偶根时可表示该边指向空节点
    array<int, BASE> nexts = { };
};
```

### 可拓展父节点查询

PAM 构建过程中，当在位置为 $loc$ 的位置插入了一个字符 $ch$，应从上一个位置的所有回文后缀中找到一个回文串 $P$，使得 $P$ 的前一个字符 $\operatorname{S}[loc - \operatorname{len}(P) - 1]$ 也为 $ch$，则由 $P$ 向两侧拓展出一个字符 $ch$ 生成一个回文串。

$P$ 从上一个位置表示的回文串开始，当 $P$ 的前一个不为字符 $ch$ 时，沿 fail 链不断向上跳转，即 $P$ 不断变为他自身的非平凡最大回文后缀，直到回文串 $P$ 能够通过字符 $ch$ 向两侧 拓展出回文串结束。回文串 $P$ 所在的树上节点作为父节点拓展出新的回文串。

```cpp
// p 表示回文后缀 P 所在的树上节点
int get_father(int p, int loc, char ch)
{
    while (!(loc - pam[p].len - 1 >= 0 && s[loc - pam[p].len - 1] == ch))
    {
        p = pam[p].fail;
    }
    return p;
}
```

若 $P$ 跳转为了空串，此时节点来到偶根，长度为 $0$。则由空串尝试拓展出仅由两个字符 $\textbf{ch}$ 构成的回文串。

若 $loc - 1$ 位置的字符不为 $ch$，再次匹配失败时，应当产生一个仅由 $\textbf{ch}$ 本身构成的单字符回文串。而单字符回文串应由奇根拓展出来。因此，为方便跳转，定义偶根的失配指针指向奇根。

>   [!Note]
>
>   奇根的 fail 指针没有实际意义。

两个根节点的特殊数据应在初始化时设置。

```cpp
void init()
{
    // 另节点 0 和节点 1 分别为偶根和奇根
    pam.assign(2, Data{ });
    pam[0].fail = 1;
    pam[1].len = -1;
}
```

### 从父节点拓展回文串

找到可拓展的树上父节点之后，根据该节点相对于字符 $ch$ 的后继边找到 $loc$ 位置所代表的回文串所在的树上节点。若该后继边指向空节点，则创建新节点。

创建新节点时，新节点所表示回文串的长度为父节点所表示回文串的长度加 $2$。

建立新节点的 fail 指针时，若新节点表示的回文串为单个字符形成的回文串，其最大非平凡回文后缀是空串，因此指向偶根节点；否则，父节点表示的回文串的所有非平凡回文后缀中寻找能够拓展字符 $ch$ 的回文后缀，形成的回文串即为 $loc$ 位置的回文串的最大非平凡回文后缀。

```cpp
void add(char ch)
{
    // turn 函数将字符映射为整数下标
    int index = turn(ch);
    // 新插入字符的下标等于之前插入字符的数量
    int loc = prefix.size();
    prefix += ch;

    // last 表示 loc - 1 位置的字符串所在的树上节点
    int father = get_father(last, loc, ch);
    
    // 拓展出的回文串节点不存在则新建
    if (pam[father].nexts[index] == 0)
    {
        // 创建当前位置的字符串所在的树上节点
        int cur = pam.size();
        pam.emplace_back();
        pam[father].nexts[index] = cur;
        // 设置新节点信息
        pam[cur].len = pam[father].len + 2;
        if (pam[cur].len == 1)
        {
            pam[cur].fail = 0;
        }
        else
        {
            int fail_father = get_father(pam[father].fail, loc, ch);
            pam[cur].fail = pam[fail_father].nexts[index];
        }
    }
    
    // 最后记录当前节点为最后插入的回文串所在的树上节点
    last = pam[father].nexts[index];
}
```

>   [!Tip]
>
>   由于 PAM 每次处理新字符只依赖已处理前缀信息，因此支持在模式串的结尾的**在线增量构建**。每次追加新字符时只需从上一个字符出发延 fail 链查找位置建立节点即可。

# 回文自动机例题

## [循环子串](https://qoj.ac/contest/1440/problem/7876/statement/zh_cn)

### 题意

对于长度为 $n$ 的字符串 $s$，将其首尾拼接成为一个循环字符串，定义其长度不超过 $n$ 的子串为循环字子串。对于这个循环字符串，找到他的回文循环子串集合。对于集合中的所有回文串 $p$，求 $\sum (\operatorname{count}(p) ^ 2 \times \operatorname{len}(p))$。

### 解法

将 $s$ 拓展为两份，另 $s' = s + s$。其中 $s'$ 中起始点在前半部分且长度小于等于 $n$ 的回文串即为目标回文循环子串。

可将 $s$ 视为 $s'$ 的后半部分，对 $s$ 和 $s'$ 分别建立回文自动机进行维护。由于 $s$ 是 $s'$ 的真子集，则对 $s$ 建立的回文自动机的树上节点在对 $s'$ 建立的回文自动机上也一定存在，且出现次数只多不少。那么可用对两个回文自动机同时进行 DFS，从 $s'$ 的回文自动机中减去 $s$ 的回文自动机中回文串的出现次数，就只剩下了起始点在前半部分的回文串。

最后只需要遍历剩下的回文串信息，若回文串的长度小于等于 $n$，就将其出现次数和长度计算后累加至答案。

### 代码

回文自动机（额外维护回文串出现次数）

```cpp
struct PAM
{
    struct Data
    {
        int count = 0;
        int len   = 0;
        int fail  = 0;
        array<int, 10> nexts = { };
    };

    int last = 0;
    string pattern;
    vector<Data> pam;

    PAM(const string& s)
    {
        pam.assign(2, { });
        pam[0].fail = 1;
        pam[1].len = -1;

        for (char c : s)
        {
            add(c);
        }
        accumulate_count();
    }

    void add(char ch)
    {
        int loc = pattern.size();
        pattern += ch;

        int index = turn(ch);

        int father = get_fail(last, loc, ch);
        if (pam[father].nexts[index] == 0)
        {
            int cur = pam.size();
            pam.emplace_back();
            pam[father].nexts[index] = cur;

            pam[cur].len = pam[father].len + 2;
            if (pam[cur].len == 1)
            {
                pam[cur].fail = 0;
            }
            else
            {
                int fail = get_fail(pam[father].fail, loc, ch);
                pam[cur].fail = pam[fail].nexts[index];
            }
        }

        last = pam[father].nexts[index];
        ++pam[last].count;
    }

    int get_fail(int father, int loc, char ch)
    {
        int fail = father;
        while (!(loc - pam[fail].len - 1 >= 0 && pattern[loc - pam[fail].len - 1] == ch))
        {
            fail = pam[fail].fail;
        }
        return fail;
    }

    // 将各位置生成的最大回文串出现次数累加至其所有回文后缀
    void accumulate_count()
    {
        for (int i = pam.size() - 1; i >= 0; --i)
        {
            pam[pam[i].fail].count += pam[i].count;
        }
    }

    int turn(char c)
    {
        return c - '0';
    }
};
```

业务流程代码

```cpp
void solve()
{
    int n;
    cin >> n;
    string s;
    cin >> s;

    PAM pam1(s + s), pam2(s);

    auto dfs = [&pam1, &pam2](this auto&& dfs, int x, int y) -> void
    {
        if (x != 0 && x != 1)
            pam1.pam[x].count -= pam2.pam[y].count;

        for (int i = 0; i < 10; ++i)
        {
            if (pam2.pam[y].nexts[i] == 0)
                continue;
            dfs(pam1.pam[x].nexts[i], pam2.pam[y].nexts[i]);
        }
    };

    dfs(0, 0);
    dfs(1, 1);

    int64_t ans = 0;
    int64_t mod = 998'244'353;
    for (auto& [count, len, fail, nexts] : pam1.pam)
    {
        if (len <= 0 || len > n)
            continue;
        ans = (ans + int64_t(count) * count % mod * len % mod) % mod;
    }
    cout << ans << '\n';
}
```

