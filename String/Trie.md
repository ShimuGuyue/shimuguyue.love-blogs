---
title: 字典树
description: 字符串集合字典维护结构
category: 字符串
tags: [树, 前缀, 字典树, 前缀树, Trie]
update_time: 2026-08-28
file_path: String/Trie
---

# 字典树

**字典树**（Trie）是一种树形数据结构，通过提取若干字符串序列的公共前缀，维护出一个字典集合。由于其前缀存储方案，因此也成为**前缀树**（Trie）。

对于问题中的字符集大小 $base$，Tire 的每个节点通过 $base$ 条边指向字典中后续存在的字符序列所在的节点。Trie 上从根结点出发一路向下，到达任意一个节点处**依次经过的字符序列**表示一个前缀。

## 插入字符串

向 Trie 中插入字符串时，遍历字符串中的每个字符，同时从根节点出发，每遍历一个字符，树上节点的位置通过对应的边下移一层，最后当字符串到达末尾时，在对应节点添加一个结束标记。

插入的时间复杂度完全取决于字符串长度，为 $O(n)$。

```cpp
struct Node
{
    array<int, 26> nexts; // 通过边指向的下一个节点编号（0 表示不存在）
    int count_end;        // 节点处字符串结束标记的个数  
};
vector<Node> trie(1); // trie[0] 节点作为根节点

void insert(string s)
{
    int node = 0;
    for (char c : s)
    {
        // turn 函数用于将字符映射成base内的整数值
        int index = turn(c);
        // 对应边不存在时创建新节点并连边
        if (trie[node].nexts[index] == 0)
        {
            trie[node].nexts[index] = trie.size();
            trie.emplace_back();
        }
        // 通过对应边移动到下一层节点
        node = trie[node].nexts[index];
    }
    // 字符串集合中可能存在重复，所以结束标记用int而非bool
    ++trie[node].count_end;
}
```

## 检索字符串

在 Trie 中检索指定字符串时，同样从根节点出发，逐字符依次下行，若 Trie 上的边不足以走到字符串结尾，则该字符串不存在于字典中。

检索的时间复杂度完全取决于字符串长度，为 $O(n)$。

```cpp
int count(string s)
{
    int node = 0;
    for (char c : s)
    {
        int index = turn(c);
        // 走不到字符串结尾直接返回
        if (trie[node].nexts[index] == 0)
            return 0;
        node = trie[node].nexts[index];
    }
    // 返回字符串结尾节点的结束标记计数
    return trie[node].count_end;
}
```

## 根据前缀检索字典串

对于给定的字符串 $s$，求在字典中以其为前缀的所有字符串，这是 Trie 的最主要应用。

对于 Trie 的每个节点，额外维护一个变量 `count_pass`，用于记录有多少个字符串经过当前节点，即以当前节点表示的前缀串为前缀。插入字符串时，对于经过的每个节点，累加其 `count_pass`。

检索前缀时，与检索整个字符串逻辑相同，首先找到前缀串对应的节点，返回该节点的 `count_pass` 即可。若字典中无该前缀串则返回 $0$。

检索的时间复杂度完全取决于字符串长度，为 $O(n)$。

```cpp
int count_pre(string s)
{
    int node = 0;
    for (char c : s)
    {
        int index = turn(c);
        if (trie[node].nexts[index] == 0)
            return 0;
        node = trie[node].nexts[index];
    }
    return trie[node].count_pass;
}
```

若要列出所有以 $s$ 为前缀的字典串，则以 $s$ 所在节点开始向下进行 DFS 回溯，每遇到一个 `count_end` 不为 $0$ 的节点，就将其记录即可。

## 删除字符串

需要从 Trie 中删除指定字符串时，进行插入的**反操作**。

首先在 Trie 中找到对应字符串的结束节点，若节点不存在，则直接结束删除操作。

找到结束节点后，首先将该节点的 `count_end` 减一，然后从该节点一路向上直到根节点，将路径上所有节点的 `count_pass` 减一。

当路径上某个节点的 `count_pass` 在操作之后变为 $0$，则说明该节点表示的前缀串在字典中不复存在。可将该节点删除，并将父节点对应的边设为空。但使用 `vector` 存储节点时，节点被删除后无法被复用，因此一般不实际删除节点，而是通过将 `count_pass` 置为 $0$ 的行为实现**懒删除**。

使用懒删除时，需将检索字符串中判断边不存在的代码由 `if (trie[node].nexts[index] == 0)` 修改为 `trie[node].nexts[index] == 0 || trie[node].nexts[index].count_pass == 0)`。

```cpp
void erase(string s)
{
    int node = 0;
    // 首先尝试找到指定字符串结束节点
    for (char c : s)
    {
        int index = turn(c);
        if (trie[node].nexts[index] == 0)
            return;
        node = trie[node].nexts[index];
    }
    if (trie[node].count_end == 0)
        return;

    // 确认字符串存在后进行删除操作
    --trie[node].count_end;
    --trie[0].count_pass;
    node = 0;
    for (char c : s)
    {
        int index = turn(c);
        node = trie[node].nexts[index];
        --trie[node].count_pass;
    }
}
```

