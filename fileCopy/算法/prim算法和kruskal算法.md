---
layout:     post
title:      最小生成树
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




> 图论中的一种算法，可在加权连通图里搜索最小生成树。意即由此算法搜索到的边子集所构成的树中。不但包括了连通图里的全部顶点（英语：Vertex (graph theory)），且其全部边的权值之和亦为最小。

## kruskal算法
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

local trees = {}

local checkTwoNodeBelongToOneTree = function(node1, node2)
    local tree1, i1, tree2, i2
    for i, v in ipairs(trees) do
        for j, k in ipairs(v) do
            if k == node1 then
                tree1 = v
                i1 = i
            end
        end
    end
    for i, v in ipairs(trees) do
        for j, k in ipairs(v) do
            if k == node2 then
                tree2 = v
                i2 = i
            end
        end
    end
    if not tree1 then
        print(node1,node2)
    end
    for i, v in ipairs(tree1) do
        if v == node2 then
            return true
        end
    end
    local merTree
    if i1 > i2 then
        table.move(tree1, 1, #tree1, #tree2 + 1, tree2)
        merTree = i1
    else
        table.move(tree2, 1, -1, -1, tree1)
        merTree = i2
    end
    table.remove(trees, merTree)
    return false
end

local function checkNode(node)
    for i, v in ipairs(nodes) do
        if v == node then
            return false
        end
    end
    table.insert(nodes, node)
    return true
end

local function increaseTree(node, addNode)
    for i, v in ipairs(trees) do
        for j, k in ipairs(v) do
            if k == node then
                table.insert(v, addNode)
                return
            end
        end
    end
end

---@param graph graph
local function minTree(graph)
    while #graph > 0 do
        local v = graph[1]
        local checkNode1 = checkNode(v.node[1])
        local checkNode2 = checkNode(v.node[2])
        if checkNode1 and checkNode2 then
            trees[#trees + 1] = {}
            table.insert(trees[#trees], v.node[1])
            table.insert(trees[#trees], v.node[2])
        elseif checkNode1 or checkNode2 then
            if checkNode1 then
                increaseTree(v.node[2], v.node[1])
            else
                increaseTree(v.node[1], v.node[2])
            end
        end
        if checkNode1 or checkNode2 or not checkTwoNodeBelongToOneTree(v.node[1], v.node[2]) then
            table.insert(result, v)
        end
        table.remove(graph, 1)
    end
end

minTree(graph)
```

## prim算法
和Kruskal算法类似，Prim算法也是使用的贪心算法。它的计算过程和Dijkstra算法非常的类似，在这里我们先学习prim算法，后面再讲述Dijkstra算法。
为了有效地实现prim算法，需要一种快速的方法来选择一条新的边来加入到集合A中边所构成的树里。
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
local nodes = {} --已经添加进去的点
local notAddNodes = { "A", "B", "C", "D", "E", "F", "G" }
local function checkNode(node)
    for i, v in ipairs(nodes) do
        if v == node then
            return false
        end
    end
    return true
end

local getNodeMinEdge = function(node)
    for i, v in ipairs(graph) do
        if v.node[1] == node or v.node[2] == node then
            local anotherNode
            if v.node[1] == node then
                anotherNode = v.node[2]
            else
                anotherNode = v.node[1]
            end
            if checkNode(anotherNode) then
                return anotherNode, v
            end
        end
    end
    return nil, nil
end

local getTheMinEdge = function()
    local edges = {}
    for i, v in ipairs(nodes) do
        local anotherNode,nodeData = getNodeMinEdge(v)
        if anotherNode then
            table.insert(edges, {anotherNode,nodeData})
        end
    end
    table.sort(edges,function (a,b)
        return a[2].value < b[2].value
    end)
    return edges[1][1],edges[1][2]
end

local removeAddNodes = function(node)
    for i, v in ipairs(notAddNodes) do
        if v == node then
            table.remove(notAddNodes, i)
            break
        end
    end
end
---@param graph graph
local function minTree(graph)
    table.insert(nodes, "A")
    removeAddNodes("A")
    while #notAddNodes > 0 do
        local node, nodeData = getTheMinEdge()
        table.insert(result, nodeData)
        table.insert(nodes, node)
        removeAddNodes(node)
    end
end

minTree(graph)
```