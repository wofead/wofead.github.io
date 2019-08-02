---
layout:     post
title:      最短路径算法
subtitle:   prim和kruskal算法
date:       2019-8-2
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - arithmetic
    - graph

---

### 目录
1. kruskal算法
2. prim算法

> 认真的人才能取得很好的成绩。
> 算两个点的最短路径可以先通过最短路径算法去除环，然后通过广度优先搜索来得到任意两个点的最短路径.

## kruskal算法
图论中的一种算法，可在加权连通图里搜索最小生成树。意即由此算法搜索到的边子集所构成的树中。不但包括了连通图里的全部顶点（英语：Vertex (graph theory)），且其全部边的权值之和亦为最小。
```lua

```

## prim算法
每次找到权值最小的边，然后进行判断是否已经形成通路，没有则添加这条边，否则不添加。
```lua
graph = {
    { value = 7, name = "ab", node = { "A", "B" } },
    { value = 8, name = "bc", node = { "B", "C" } },
    { value = 5, name = "ad", node = { "A", "D" } },
    { value = 9, name = "bd", node = { "D", "B" } },
    { value = 7, name = "be", node = { "E", "B" } },
    { value = 5, name = "ce", node = { "C", "E" } },
    { value = 15, name = "de", node = { "D", "E" } },
    { value = 6, name = "df", node = { "D", "F" } },
    { value = 8, name = "fe", node = { "F", "E" } },
    { value = 9, name = "eg", node = { "E", "G" } },
    { value = 11, name = "fg", node = { "F", "G" } }
}

table.sort(graph, function(a, b)
    return a.value < b.value
end)

local result = {}
local nodes = {}

local function checkNode(node)
    for i, v in ipairs(nodes) do
        if v == node then
            return false
        end
    end
    table.insert(nodes, node)
    return true
end

---@param graph graph
local function minTree(graph)
    while #graph > 0 do
        local v = graph[1]
        if checkNode(v.node[1]) or checkNode(v.node[2]) then
            table.insert(result, v)
        end
        table.remove(graph, 1)
    end
end

minTree(graph)
```
