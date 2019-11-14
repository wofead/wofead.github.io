---
layout:     post
title:      Lua 学习 chapter14
subtitle:   chapter14
date:       2019-6-17
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 数据结构
2. 链表
3. 队列以及双端队列
4. 反向表
5. 集合和包
6. 字符串缓冲区
7. 图像


> What is the cost of lies? It's not that we'll mistake them for the truth. The real danger is that if we hear enough lies, then we no longer recognize the truth at all. What can we do then? What else is left but to abandon even the hope of truth and content ourselves instaed with stories? In these stories, it doesn't matter who the heroes are. All we want to know is: who is to blame? No friends. Or, at least, not important ones.

## 数据结构

在lua中我们可以通过使用表来实现各种数据结构。
1. 数组
2. 矩阵和多位数组 --通过insert和嵌套表
3. 链表 --使用表来作为节点
4. 队列和双端队列
5. 反向表
6. 集合和包  --insert 和 remove
7. 字符串缓冲区 --**在这里注意大量连接字符串的时候使用table.cancat(),把字符串放到表中进行链接。第二个参数为在链接的时候插入什么。**
8. 图像

## 链表

在下面的这段代码只是简单的实现了链表，我们可以发现，它的头是没有存放数据的，仅仅是代表着这个list的头，如果不使用头插入的方法的话插入也非常的麻烦，但是lua表结构非常强大，我们可以在head里面存上尾node的引用。

```lua
---@type table<string, table>
local list = {next = nil, value = nil}

list.next = {next = {next = nil, value = 2},value = 1}

local l = list.next
while l do
    print(l.value)
    l = l.next
end

--带尾索引的list，将尾索引放在head中
local node = {next = nil,value = 1}
list.endNode.next = node

list.endNode = node

node = {next = nil,value = 2}

list.endNode.next = node

list.endNode = node

local l = list.next
while l do
    print(l.value)
    l = l.next
end
```

## 队列以及双端队列

队列的意思含义是尾部插入，头部移除。在lua中一种简单的实现就是通过insert和remove函数来实现队列就可以了。

```lua
local queue = {}

table.insert(queue,#queue + 1,1)
table.insert(queue,#queue+ 1,2)
table.insert(queue,#queue+ 1,3)

table.remove(queue,1)
```

不过上面的这种方式开销应该是比较大的，就像我们删除数组中的一个元素，后面的元素不还要移动前面么，这样子开销真的不小，每次插入到最后一个，遍历插入，开销应该也不小。

**在这里也提示我，到时候看一下lua insert和remove函数的实现。**

在这里还有另一种实现方法，有点类似于c语言中的利用数组来实现个一个队列，通过表示，首个元素和尾元素的位置来控制增加和删除，在c中，数组是存在大小的，满了就不能添加了，这个时候可以选择提示队列已满，也可以选择扩容，然后把元素重新放到新的队列中。

看了别人的实现，发现还是比较出彩的，通过使用first和end两个索引来进行双端的push和pop，然后，直接使用索引作为键值进行存储数据，在设计上，通过first减和last减进行相反方向数据的插入，当然这里让first和last的差值只相差一也是判断queue是否为空的关键。

```lua
---return table{string,int}
function listNew()
    return {first = 0, last = -1}
end

---param list table{string,int}
function pushFirst(list, value)
    local first = list.first - 1
    list.first = first
    list[first] = value
end

function pushLast(list, value)
    local last = list.last +1
    list.last = last
    list[last] = value
end

function popFirst(list)
    local first = list.first
    if first > list.last then
        print("list is empty!")
        return
    end
    local value = list[first]
    list[first] = nil
    list.first = list.first + 1
    return value
end

function popLast(list)
    local last = list.last
    if list.first > last then
        print("list is empty!")
        return
    end
    local value = list[last]
    list[last] = nil
    list.last = list.last - 1
    return value
end

local queue = listNew()

pushFirst(queue,1)
pushFirst(queue,2)
pushFirst(queue,3)
pushLast(queue,-1)
pushLast(queue,-2)
pushLast(queue,-3)
popFirst(queue)
popFirst(queue)
```

## 反向表

即正向索引和反向索引都能取到与之对应的值。

```lua
local reverseTable = {"a","b","c","d","e"}

for i, v in ipairs(reverseTable) do
    reverseTable[v] = i
end

```

## 集合和包

所谓的集合就是每个元素在这个集合中只存在一份数据，而包，意思是可以存在多分数据。

集合：使用表来表达，就是每次插入值的时候，直接把值作为键值，值设置为true就行了，remove直接设置为nil。

包：使用表来表达，就是每次插入值的时候，直接把值作为键值，值进行自增（如果原来存在值，否则值为1），remove的时候，值进行减一，如果减到了0，则设置为nil。

```lua
local set = {}

function insertSet(set, va)
    set[va] = true
end

function removeSet(set, val)
    set[val] = nil
end

function getSet(set)
    for i, v in pairs(set) do
        print(i)
    end
end

insertSet(set,1)
insertSet(set,2)
insertSet(set,2)
insertSet(set,3)

removeSet(set,3)
removeSet(set,2)
getSet(set)

local bag = {}

function insertBag(bag, va)
    if bag[va] then
        bag[va] = bag[va] + 1
    else
        bag[va] = 1
    end
end

function removeBag(bag, val)
    if not bag[val] then
        print("the bag doesn't contain this val")
        return
    end
    bag[val] = bag[val] - 1
    if bag[val] == 0 then
        bag[bag] = nil
    end
end

function getBag(bag)
    for i, v in pairs(bag) do
        for j = 1, v do
            print(i)
        end
    end
end

insertBag(bag,1)
insertBag(bag,2)
insertBag(bag,2)
insertBag(bag,2)
insertBag(bag,3)

removeBag(bag,3)
removeBag(bag,2)
getBag(bag)

print(bag)
```

## 字符串缓冲区

我们在写程序的时候经常使用“..”的方式来连接字符串，其实这样做在连接大量的字符串的时候会存在很大的问题，因为每连接一次都会产生一份新的内存空间用来存放产生的字符串，每次产生新的字符串的时候程序都会去遍历整个存放字符串的空间，看是否需要开辟新的内存空间来存放新的字符串。

```lua
local buff = ""
for line in io.lines() do
	buff = buff .. line .. "\n"
end
```

至于上面的问题，我说的可能并不是很对，这里摘抄一下原话：
> 假设每行有20个字节，当我们读取了大概2500行后，buff就会变成一个50KB大小的字符串，然后从buff中复制50000字节中到这个新的字符串中。这样，对于后续的每一行，lua语言都需要移动大概50KB且还在不断增长的内存。因此，该算法的时间复杂度是二次的。在读取了100行（仅仅2KB）以后，lua语言就已经移动了至少5MB内存。当然并不是仅仅lua存在这个问题，对于只要是字符串是不可变值，就会出现类似的问题，其中java也是这个样子的。

所以我们通常可以字符串缓存空间来避免这个问题。然后使用table.concat函数将列表中的所有字符串连接起来并返回连接后的结果。

```lua
local tabBuff = {}
for line in io.lines() do
	tabBuff[#tabBuff + 1] = line
end

local s = table.concat(tabBuff,"\n") .. "\n"
```

## 图形

之前一直有用lua进行算法的学习，当初只是怎么方便算法对图的应用就怎么来，没有考虑那么多，当然，今天要讨论一下怎么构造一个合格且通用的图结构呢？

```lua
--图的构建和遍历
local function name2node(graph,name)
    local node = graph[name]
    if not node then
        --node doesn't exist
        node = {name = name, adj = {}}
        graph[name] = node
    end
    return node
end

local function readGraph(fileName)
    local f = io.open(fileName)
    io.input(f)
    local graph = {}
    for line in io.lines() do
        local nameFrom, nameTo = string.match(line,"(%S+)%s+(%S+)")
        local from = name2node(graph, nameFrom)
        local to = name2node(graph,nameTo)
        table.insert(from.adj,to)
    end
    return graph
end

local tempGraph = readGraph("graph")

local function traceGraph(graph)
    for i, v in pairs(graph) do
        for j, s in ipairs(v.adj) do
            print(v.name,s.name)
        end
    end
end

traceGraph(tempGraph)
```

图的深度优先搜索和广度优先搜索。

深度优先搜索利用的数据结构是栈，深度优先搜索使用的是队列。

这里我们使用深度优先搜素来找到you到anuj的路线。

```lua
--深度优先搜索和广度优先搜索都只是遍历图的
local function wideTrace(graph, startNodeName)
    local visited = {}
    local queue = {}
    local curNode = graph[startNodeName]
    table.insert(queue, curNode)
    while #queue > 0 do
        local theVisitNode = table.remove(queue, 1)
        visited[theVisitNode.name] = true
        print(theVisitNode.name)
        for i, v in ipairs(theVisitNode.adj) do
            if not visited[v.name] then
                table.insert(queue,v)
            end
        end
    end
end

wideTrace(tempGraph, "you")

local function deepTrace(graph, startNodeName)
    local visited = {}
    local stack = {}
    local curNode = graph[startNodeName]
    table.insert(stack, curNode)
    while #stack > 0 do
        local theVisitNode = table.remove(stack, #stack)
        visited[theVisitNode.name] = true
        print(theVisitNode.name)
        for i, v in ipairs(theVisitNode.adj) do
            if not visited[v.name] then
                table.insert(stack,v)
            end
        end
    end
end

deepTrace(tempGraph, "you")
```

使用深度优先遍历递归每条路线，寻找通路。

```lua
local function findPath(graph,curr, to, path, visited)
    path = path or {}
    visited = visited or {}
    if visited[curr] then
        return nil
    end
    visited[curr] = true
    path[#path + 1] = curr
    if curr == to then
        return path
    end
    for i,node in ipairs(graph[curr].adj) do
        local p = findPath(graph,node.name, to , path, visited)
        if p then
            return p
        end
    end

    table.remove(path)
end

local thePath = findPath(tempGraph,"you","anuj")
```

在这里我们还可以给边添加距离的信息，然后通过使用dijkstra算法来计算两个点之间的最短距离。

在添加value值的时候，注意一点，那就是别忘记标注究竟是谁到目标点的距离。

```lua
local function name2node(graph, name)
    local node = graph[name]
    if not node then
        --node doesn't exist
        node = { name = name, adj = {}, label = tonumber(string.sub(name,2)), value = {} }
        graph[name] = node
    end
    return node
end

local function readGraph(fileName)
    local f = io.open(fileName)
    io.input(f)
    local graph = {}
    for line in io.lines() do
        local nameFrom, len, nameTo = string.match(line, "(%S+)%s+(%S+)%s+(%S+)")
        local from = name2node(graph, nameFrom)
        local to = name2node(graph, nameTo)
        --一定要加上label，否则不知道谁到这个节点的长度
        to.value[from.label] = len
        table.insert(from.adj, to)
    end
    return graph
end

local tempGraph = readGraph("graph")


```

Dijkstra算法，又复习了一下，现在还是存在一些疑问，不过先把我实现的代码贴到这里，一会儿根据代码来分析。

```lua
local function printArc(arc)
    for i, v in ipairs(arc) do
        for j, k in ipairs(v) do
            if k == math.maxinteger then
                io.write("\t∞")
            else
                io.write("\t" .. k)
            end
        end
        io.write("\n")
    end
end

-- 每个节点的访问信息，visit：是否已经访问，path：路径是什么，value：值是多少
local function dijkstra(graph, startNode)
    local arc = {}
    --初始化邻接矩阵
    for i = 1, 6 do
        for j = 1, 6 do
            if not arc[i] then
                arc[i] = {}
            end
            arc[i][j] = math.maxinteger
        end
    end

    --根据图里面的信息赋初值
    for i, v in pairs(graph) do
        for i, node in ipairs(v.adj) do
            arc[v.label][node.label] = tonumber(node.value[v.label])
        end
        arc[v.label][v.label] = 0
    end

    printArc(arc)

    local dis = {}
    startNode = graph[startNode]
    --初始化dis数组
    for i, v in ipairs(arc[startNode.label]) do
        local nodeInfo = {value = 0, visit = false, path = ""}
        if v == 0 then
            nodeInfo.visit = true
            nodeInfo.path = startNode.name .. "---->" .. startNode.name
        elseif v < math.maxinteger then
            nodeInfo.value = v
            nodeInfo.path = startNode.name .. "---->" .. graph["v"..i].name
        else
            nodeInfo.value = math.maxinteger
        end
        dis[i] = nodeInfo
    end
    local count = 1
    local temp = 0
    while count ~= 6 do
        temp = 0
        local min = math.maxinteger
        for i, v in ipairs(dis) do
            if not v.visit and v.value < min then
                min = v.value
                temp = i
            end
        end
        -- 已经全部寻找完了
        if temp == 0 then
            break
        end
        dis[temp].visit = true
        count = count + 1
        for i = 1, 6 do
            if not dis[i].visit and arc[temp][i] ~= math.maxinteger and (dis[temp].value + arc[temp][i] < dis[i].value) then
                dis[i].value = dis[temp].value + arc[temp][i]
                dis[i].path = dis[temp].path .. "-->v" .. tostring(i)
            end
        end
    end

    for i, v in ipairs(dis) do
        if v.visit == false then
            print("not a path can access v" .. i)
        else
            print(v.path .. " = " .. v.value)
        end
    end
end

dijkstra(tempGraph, "v1")

```

Dijkstra算法采用的是一种贪心的策略，核心是一个dis的数组，这个数组放着开始节点到其它节点的信息，距离，是否访问和如何访问的路径，每次我们在dis中（出去访问过的点）寻找一个最小的值，然后将该值标记为已经访问，并以该点为起始点遍历他能到的点，算出距离和起始点到该点的距离进行比较，如果小，则更新value值和path。