---
title: 线段树
description: 可汇总信息的区间维护结构
category: 数据结构
tags: [树, 线段树, 区间问题]
update_time: 2026-08-31
file_path: DataStructure/SegmentTree
---

# 线段树

线段树是一种用于解决**可汇总**信息（可重复贡献信息、可差分信息等，例如区间总和）的**区间查询和修改**操作的数据结构。

线段树通过将区间不断**分段**，将大区间的信息划分为小区间信息的汇总。

>   一般情况下，将线段树建为一颗二叉树。实际可根据操作系统底层等因素改用其它的多叉树。

## 初始化建树

线段树上的每个节点维护一段区间的汇总信息，父节点信息由所有子节点汇总得到，其中根节点维护整个区间的汇总信息。

每个节点将区间长度平分为两半分别交由左右两个子节点维护，直到某些节点仅维护一个位置的信息时即为叶子节点。

对于一个长度为 $n$ 的序列，构造的二叉树结构的线段树中叶子节点共有 $n$ 个，维护信息所使用的节点总数为 $2n - 1$。

---

初始化线段树时，首先创建一个根节点其维护整个区间。然后从根节点出发，不断将节点维护区间二分之后分别递归建立左右子树，直到区间长度降为 $1$ 即到达叶子节点。

将单点数据填入叶子节点之后，不断向上回溯，不断将两个子节点的区间值汇总到父节点，直到到达根节点。

父节点由子节点处信息汇总得到大区间信息的操作称为 push up。

```cpp
struct Node
{
    int data;   // 维护的区间汇总值
    int seg_l;  // 维护的区间端点
    int seg_r;
    int node_l; // 子节点编号
    int node_r;
};

vector<Node> tree(1);
```

```cpp
void build(int l, int r, int node, vector<int>& arr)
{
    // 记录每个节点维护的区间位置
    tree[node].seg_l = l;
    tree[node].seg_r = r;
    // 到达叶子节点时回溯
    if (l == r)
    {
        tree[node].data = arr[l];
        return;
    }
    // 为非叶子节点创建两个子节点并递归建立子树
    tree[node].node_l = tree.size();
    tree.emplace_back();
    tree[node].node_r = tree.size();
    tree.emplace_back();

    int mid = (l + r) / 2;
    build(l,     mid, tree[node].node_l, arr);
    build(mid + 1, r, tree[node].node_r, arr);
    // 从子树汇总信息
    push_up(node);
}

build(0, n - 1, 0, arr);
```

```cpp
void push_up(int node)
{
    int l = tree[node].node_l;
    int r = tree[node].node_r;
    tree[node].data = tree[l].data + tree[r].data;
}
```

## 单点修改

进行单点修改操作时，只需要从根节点出发，不断判断目标点被哪一个子节点维护，然后一路向下直到叶子节点，即维护该位置信息的唯一节点。

对叶子节点信息进行修改，然后一路回溯，不断将子节点的信息汇总到父节点，直至到达根节点即完成一次修改。

例如操作：在 $index$ 位置上加上值 $data$。

```cpp
void update(int index, int data, int node)
{
    if (tree[node].seg_l == tree[node].seg_r)
    {
        tree[node].data += data;
        return;
    }
    // 根据目标位置所在节点访问下层节点
    int mid = (tree[node].seg_l + tree[node].seg_r) / 2;
    if (index <= mid)
        update(index, data, tree[node].node_l);
    else
        update(index, data, tree[node].node_r);
    // 将子节点修改后的信息汇总到父节点
    push_up(node);
}
```

每次进行单点修改的时间复杂度为 $O(\log n)$。

>   [!Note]
>
>   线段树的主要用法是**区间**的修改询问，因此一般用区间修改操作代替单点修改操作。

## 区间修改 | 懒标记

若要进行区间修改操作，容易想到的方法是对于区间中的每一个点进行单点修改。但这种方式的时间复杂度为 $n \log n$，时间消耗过于繁重，需要进行优化。

为了解决这个问题，线段树引入**懒标记**的概念。即当一个节点所维护的区间完全被要修改的区间所包含时，不再向下递归，而是在当前节点记录一个标记，指示该区间尚未进行的修改操作，同时根据区间修改的信息，修改当前节点的数据。等到子节点的数据需要被使用时，才根据懒标记对子节点进行修改，修改对应节点的数据，将当前节点的懒标记清除并下传到子节点。

将父节点的懒标记下传到子节点并修改子节点信息的操作称为 push down。

例如操作：将区间 $[l,\ r]$ 内的所有位置的值加上 $data$。

```cpp
// 将 l, r 定义为要修改的区间在当前节点维护的区间内的截断部分
void update(int l, int r, int data, int node)
{
    // 若当前节点被区间完全包含，设置懒标记，终止递归
    if (tree[node].seg_l == l && tree[node].seg_r == r)
    {
        int len = tree[node].seg_r - tree[node].seg_l + 1;
        // 根据懒标记修改当前节点汇总值
        tree[node].data += data * len;
        // 累加懒标记
        tree[node].tag += data;
        return;
    }
    // 递归修改之前首先将已有的懒标记下传
    push_down(node);
    // 递归修改子节点
    int mid = (tree[node].seg_l + tree[node].seg_r) / 2;
    if (r <= mid)     // 整个区间全在左子树
        update(l, r, data, tree[node].node_l);
    else if (l > mid) // 整个区间全在右子树
        update(l, r, data, tree[node].node_r);
    else              // 区间分属于左右两子树
        update(l,     mid, data, tree[node].node_l),
        update(mid + 1, r, data, tree[node].node_r);
    // 将修改后的信息汇总
    push_up(node);
}
```

```cpp
void push_down(int node)
{
    // 将父节点懒标记下传并修改子节点
    for (int child : { tree[node].node_l, tree[node].node_r })
    {
        // 叶子节点没有子节点，懒标记无需下传
        if (tree[node].seg_l == tree[node].seg_r)
            break;
        int len = tree[child].seg_r - tree[child].seg_l + 1;
        // 先根据新的懒标记修改子节点信息，再累加子节点懒标记
        tree[child].data += len * tree[node].tag;
        tree[child].tag += tree[node].tag;
    }
    // 将懒标记设为一个参与运算没有任何影响的值，
    // 以复用具有懒标记的代码逻辑，实现伪删除
    tree[node].tag = 0;
}
```

## 区间查询

进行区间查询操作时，仍然从根节点出发，不断向下递归找到恰好覆盖住对应区间的若干个节点，将这些节点的信息汇总到一起即得到答案。

若向下递归之前当前节点存在未被处理的懒标记，应当先将其下传，再进行后续操作。

```cpp
// 将 l, r 定义为要修改的区间在当前节点维护的区间内的截断部分
int query(int l, int r, int node)
{
    if (tree[node].seg_l == l && tree[node].seg_r == r)
        return tree[node].data;

    push_down(node);
    int mid = (tree[node].seg_l + tree[node].seg_r) / 2;
    if (r <= mid)
        return query(l, r, tree[node].node_l);
    else if (l > mid)
        return query(l, r, tree[node].node_r);
    else
        return query(l,     mid, tree[node].node_l)
             + query(mid + 1, r, tree[node].node_r);
}
```

