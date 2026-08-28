---
title: 并查集
description: 不相交集合的关系维护
category: 数据结构
tags: [并查集, 集合]
update_time: 2026-08-28
file_path: DataStructure/DisjointSet
---

# 并查集

**并查集**（Disjoint Set）是一种用于处理**不相交集合合并与查询**的数据结构，常用于连通性判定等问题。

并查集的核心原理是通过一棵以根节点为代表的**树形结构**来维护集合划分，每个集合用一棵树表示，树的根节点即该集合的代表元素。

初始化时，每个节点单独在一个集合，其树的根节点为其本身。

```cpp
vector<int> fathers;

void init(int n)
{
    fathers.resize(n);
    for (int i = 0; i < n; ++i)
    {
        fathers[i] = i;
    }
}
```

## 查询集合所属

查询某个节点所在集合时，首先判断该节点是否为所在集合的根节点，如果不是，持续向上搜索祖先节点直至**根节点**处，返回根节点编号。	

```cpp
int find(int x)
{
    while (fathers[x] != x)
    {
        x = fathers[x];
    }
    return x;
}
```

### 路径压缩

查找过程可进行路径压缩处理，即将查找路径上所有节点都直接连接到集合根节点上，后续查找速度会更快。

>   [!WARNING]
>
>   路径压缩会**破坏树的结构**，在需要严格保持合并顺序的场景下不能使用，例如可撤销并查集。

```cpp
int find(x)
{
    // 先找到根节点
    int root = x;
    while (fathers[root] != root)
    {
        root = fathers[root];
    }
    // 将链条上每个节点直接挂载到根节点上
    while (fathers[x] != root)
    {
        int father = fathers[x];
        fathers[x] = root;
        x = father;
    }
    return root;
}
```

使用路径压缩操作时，树的形状趋向于扁平化，每次查询的平均次数很少，可视为一个可忽略的常数，因此可以采用递归的写法简化代码，一般不会有过度消耗时间或者栈溢出的风险。

```cpp
int find(x)
{
    return fathers[x] == x ? x : fathers[x] = find(fathers[x]);
}
```

## 合并不同集合

将两个节点所在集合进行合并时，首先找到各自集合的根节点，然后将其中一棵树挂载到另一颗树的根节点下即可。

```cpp
void merge(int x, int y)
{
    int set_x = find(x);
    int set_y = find(y);
    fatehers[set_x] = set_y;
}
```

### 按秩合并

若使用以上朴素合并方式，随合并次数的增加，可能会使得树的重心朝一个方向偏移，导致树高增加。

为了让树尽可能保持平衡，可以采用按秩（即树高）合并的方式，每次合并时将高度较小的树挂载到高度较高的那颗下，以尽可能减缓树高的增长速度。

```cpp
int merge(int x, int y)
{
    int set_x = find(x);
    int set_y = find(y);
    if (ranks[set_x] < ranks[set_y])
    {
        fathers[set_x] = set_y;
    }
    else
    {
        fatherx[set_y] = set_x;
        if (ranks[set_x] == ranks[set_y])
            ++ranks[set_x]; // 两集合树高一致时，被选作根节点的树高加一
    }
}
```

在不使用路径压缩的情况下，仅按秩合并会使树高不会超过为 $\log_2 n$，因此 `find` 操作的时间复杂度为 $O(n \log n)$。

在使用路径压缩的情况下，树的扁平化程度较高，一般并不需要按秩合并进行额外优化，`find` 操作的平均时间复杂度为 $O(\alpha (n))$，其中 $\alpha (n)$ 是反阿克曼函数，实际可近似看作是一个不超过 $5$ 的常数

合并操作的复杂度取决于是否采用路径压缩和按秩合并的优化。

>   [!Warning]
>
>   对于需要严格控制树节点上下级关系的场景，按秩合并无法使用。

## 维护集合大小

维护所有集合的大小时，可以额外记录一个数组 `sizes` 表示每个节点作为根节点时该集合的大小。查询节点所在集合大小时，先找到根节点，再返回维护的数据即可。

集合合并时，只需把两个集合大小之和赋值给新的根节点即可。

## 维护集合数量

维护集合数量时，可以初始记录一个 `count` 为所有节点的数量，每次进行集合合并时，集合数减一，查询时直接返回即可。
