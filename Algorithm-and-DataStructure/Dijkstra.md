---
title: Dijkstra
description: 无负权图的单源最短路径算法
category: 数据结构和算法
tags: [图论, 最短路径, 单源最短路, 贪心, Dijkstra]
update_time: 2026-08-28
file_path: Algorithm-and-DataStructure/Dijkstra
---

# Dijkstra

**Dijkstra** 算法用于求解**无负权图**的**单源最短路**。

Dijkstra 算法从一个起始点出发，基于**贪心策略**和**松弛操作**不断更新到达相邻点的最短路径。

* 松弛：对于**已确定**最短路径的所有点 $u$，遍历其相邻的所有点 $v$，尝试更新其最短路径。

* 贪心：对于**未确定**最短路径的点所有中，选择当前距离起点最近的点 $v$，确定他的最短路径。


## 朴素版 Dijkstra

**朴素版** Dijkstra 中，图的存储方式使用**邻接矩阵**，该方法更适用于**稠密图**。

算法共进行 $n$​ 次循环操作，每次**遍历所有点**，找到当前未确定最短路径的点中最短路径最短的点。

找到目标点后，用该点的最短路径尝试更新所有相邻点的最短路径，之后开启下一轮循环。

该方法使用双层循环，时间复杂度为 $O(n ^ 2)$。

```c++
vector<int64_t> dijkstra(vector<vector<int64_t>>& graph, int start)
{
    static constexpr int64_t inf = 1 << 60;

    int n = graph.size();
    vector<int64_t> visited(n, false);
    vector<int64_t> min_lens(n, inf);
    min_lens[start] = 0;

    // n 次循环，每次确定一个点的最短路径
    for (int i = 0; i < n; ++i)
    {
        int64_t min = inf;
        int node = -1;
        for (int v = 0; v < n; ++v)
        {
            // 已经确定最短路径的节点跳过，无需再更新
            if (visited[v])
                continue;
            if (min_lens[v] < min)
            {
                min = min_lens[v];
                node = v;
            }
        }
        // 如果所有可达点的最短路已全部找到，提前退出循环
        if (node == -1)
            break;
        // 将找到的节点标记为已找到最短路径
        visited[node] = true;
        // 根据新找到节点的最短路径更新其相邻点的最短路径
        for (int v = 0; v < n; ++v)
        {
            // 没有连边的两个点不可直接到达
            // 已确定最短路径的点距离不会更短（无负权图）
            if (graph[node][v] == inf || visited[v])
                continue;
            min_lens[v] = min(min_lens[v], min_lens[node] + graph[node][v]);
        }
    }
    return min_lens;
}
```

## 堆优化版 Dijkstra

**堆优化版**的 Dijkstra 中，图的存储方式使用**邻接表**，该方法更适用于**稀疏图**。

堆优化版本使用一个小根堆维护所有未确定最短路径的点的当前最短路径，直接取堆顶找到目标点。

每次拿取目标点时，先判断当前点是否已被确定最短路径。如果是，则路径不可能更短，跳过。否则尝试更新所有相邻的点的最短距离。若更新成功，则将对应点加入堆中。

该方法需遍历所有点和边，堆的大小取决于边的数量，故时间复杂度为 $O((n + m) \log m)$

```c++
// graph 中数据存储约定：graph[u][v] = w，
// 其中 u, v 为两个点，w 为两点直接连边的长度。
vector<int64_t> dijkstra(vector<vector<pair<int, int64_t>>>& graph, int start)
{
    static constexpr int64_t inf = 1 << 60;

    int n = graph.size();
    vector<int64_t> min_lens(n, inf);

    // 堆中 pair 的 first 和 second 分别存储 len 和 node，在小根堆中按 len 排序
    priority_queue<pair<int64_t, int>, vector<pair<int64_t, int>>, greater<pair<int64_t, int>>> h;
    h.push({0, start});
    while (!h.empty())
    {
        auto [len, u] = h.top();
        h.pop();
        // 如果当前点已经被确定了最短路径，跳过
        if (min_lens[u] != inf)
            continue;
        min_lens[u] = len;
        for (auto [v, w] : graph[u])
        {
            // 尝试更新相邻节点的最短路径并放入堆中
            if (len + w < min_lens[v])
                h.push({len + w, v});
        }
    }
    return min_lens;
}
```