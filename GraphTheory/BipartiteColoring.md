---
title: 二分图染色
description: 染色法判定二分图
category: 图论
tags: [DFS-BFS, 二分图, 二分图染色]
update_time: 2026-08-28
file_path: GraphTheory/BipartiteColoring
---

# 二分图染色

**二分图染色**（Bipartite Staining）是通过 DFS/BFS 对节点染色的操作，判断给定的图是否是二分图的算法。

进行染色操作时，以任意一点为起始点，将其标记为两种颜色之一 $A$ 或 $B$，然后从该点开始不断扩展，将每个节点的相邻节点染成另一种颜色。

若染色过程中发生冲突，即两个相邻节点的颜色相同，则说明该图不是二分图。否则该图是二分图。

>   [!Note]
>
>   根据二分图相邻节点颜色不同的性质可知，当图中存在**长度为奇数的环**时，一定会发生冲突；若图中只存在偶数长度的环或者不存在环时，一定不会发生冲突。
>
>   图中不存在奇数环和图是二分图互为**充要条件**。

整个判断过程需要遍历所有的点和边，故时间复杂度为 $O(n + m)$。

```cpp
auto judge() -> bool
{
    vector<int> colors(n);
    queue<int> q;
    // 不连通的图要对每一块分别染色
    for (int i = 0; i < n; ++i)
    {
        if (colors[i]) // 跳过已被染色的格子
            continue;
        colors[i] = 1;
        q.push(i);
        // 对一个连通块进行染色
        while (!q.empty())
        {
            int u = q.front();
            q.pop();
            // 默认使用邻接表存图
            for (int v : edges[u])
            {
                // 相邻两个节点同色说明判定失败
                if (colors[v] == colors[u])
                    return false;
                // 将相邻节点染上与当前节点不同颜色
                if (colors[v] == 0)
                {
                    colors[v] = colors[u] == 1 ? 2 : 1;
                    q.push(v);
                }
            }
        }
    }
    return true;
}
```
