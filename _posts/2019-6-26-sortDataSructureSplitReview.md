---
layout:     post
title:      排序、数据结构以及分治策略的复习
subtitle:   review
date:       2019-6-26
author:     Jow
header-img: img/lua-home-bg-o.jpg
catalog: 	 true 
tags:
    - 数据结构
    - review

---

### 目录
1. 排序



> I always tell my friends to review what you have studied is so important. But for me, it seems that someone tell me and i don't care.I think that new thing is more important then what have got. But the truth is what you have got is dispearing, and oneday it will not belong to you. 

## 排序
首先来回顾一下目前我所掌握到的排序算法有哪些，插入、选择、冒泡、归并、堆、二叉树、基数、计数、桶、希尔、快速。接下来从简单到复杂一个一个来实现。

1. 插入排序

正如所总结的那样，插入排序就像抓牌一样，不断地向一个已经排好序的序列插入新的元素。在插入排序的这次复习中，我发现自己忘记记录key和index，在比较的时候使用a[i]，而不是key，这个地方是存在问题的，因为a[i]在后面已经被覆盖了
```lua
local function insertSort(arr)
    for i = 2, #arr do
        local key = arr[i]
        local index = i
        for j = i - 1, 1, -1 do
            if arr[j] > key then --key not a[i]
                arr[j + 1] = arr[j]
                index = j --record j
            end
        end
        arr[index] = key
    end
end
insertSort(arr)
```

2. 选择排序

每次选择一个元素放到正确的位置。和插入排序很像，就是不用每次交换，只用找到位置后，将那个位置原来的元素和需要放在那个位置的元素进行交换就行了。

3. 冒泡排序

在上次，我简单的实现了冒泡排序，后面发现有两个方面可以优化，第一个方面为在交换的过程中如果发现这一轮都没有发生交换就可以说已经排好序了，后面的操作都是多余的，第二方面为发生交换的位置，最后一次交换的位置就是下次判断的上限。
```lua
local function bubbleSort(arr)
    local flag = false
    for i = 1, #arr do
        if flag then
            break
        end
        flag = true
        local lastSwap = #arr
        for j = 2, lastSwap do
            if arr[j] > arr[j - 1] then
                arr[j], arr[j - 1] = arr[j - 1], arr[j]
                flag = false
                lastSwap = j - 1
            end
        end
    end
end
bubbleSort(arr)
```

4. 基数排序
基数排序就是利用key值的位数进行排序，通过取不同的位来进行分桶放置，然后排序。在进行基数排序的过程中，发现基数排序一般排序的key值都是正整数，在求位数的时候可以先除再求余数的方法，注意除以0的时候，在这里做特殊处理。

5. 计数排序

和基数排序很像，计数排序需要知道最大值，然后额外最大值个数组空间和新的排序之后的空间。
```lua
local arr = { 13, 20, 16, 18, 20, 12, 15, 7 }
local sortResult = {}
--local arr = { 13, -3, -25, 20, -3, 16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7 }
local extra = {}
for i = 1, 20 do
    extra[i] = 0
end
local function countSort(arr)
    for i, v in ipairs(arr) do
        extra[v] = extra[v] + 1
    end
    for i = 2, 20 do
        extra[i] = extra[i - 1] + extra[i]
    end
    for i = 1, #arr do
        sortResult[extra[arr[i]]] = arr[i]
        extra[arr[i]] = extra[arr[i]] - 1
    end
    return sortResult
end
arr = countSort(arr)
```
6. 桶排序

桶排序是将元素通过散列函数进行分配放到不同的桶中，然后对桶里面的元素进行排序，最后输出。

7. 希尔排序
希尔排序是优化的插排，通过缩小每次排序的移动范围来达到优化。
在最内层的循环排序就是插入排序。
希尔排序就是优化过的插入排序，当然是针对于大量数据的。
```lua
local function hillSort(arr)
    local split = math.floor(#arr / 2)
    for i = split, 1, -1 do
        for j = 1 + i, #arr, i do
            local key = arr[j]
            local index = j
            for k = j - i, 1, -i do
                if arr[k] > key then --key not a[i]
                    arr[k + i] = arr[k]
                    index = k --record j
                end
            end
            arr[index] = key
        end
    end
end
hillSort(arr)
```



8. 快速排序

在复习的过程中，在注意点1中，需要注意怎么样分隔数组让所规定的值的左右分别达到需要，在注意点2中，由于key的位置已经正确，所以在递归中不需要再带上key值，否则会引起死循环。
```lua
local arr = { 13, 20, 16, 18, 20, 12, 15, 7 }

local function exchangeArr(arr, p, key, q)
    local i = p
    local j = q
    local keyValue = arr[key]
    while i < j do --attention1
        while arr[i] < keyValue do
            i = i + 1
        end
        arr[i],arr[key] = arr[key],arr[i]
        key = i
        i = i + 1
        while arr[j] > keyValue do
            j = j - 1
        end
        arr[j],arr[key] = arr[key],arr[j]
        key = j
        j = j - 1
    end
    return key
end

local quickSort
quickSort = function(arr, p, q)
    if p < q then
        local key = math.random(p, q)
        key = exchangeArr(arr, p, key, q)
        quickSort(arr, p, key - 1) --attention 2
        quickSort(arr, key + 1, q)
    end
end

quickSort(arr, 1, #arr)
```

9. 归并

注意在attention那个地方判断arr2和arr1所取的值不为空才能比较。

```lua
local arr = { 13, 20, 16, 18, 20, 12, 15, 7 }

local function mergeArr(arr, p, mid, q)
    local arr1 = {}
    local arr2 = {}
    table.move(arr, p, mid, 1, arr1)
    table.move(arr, mid + 1, q, 1, arr2)
    local k = 1
    local j = 1
    for i = p, q do
        if not arr2[j] or arr1[k] and arr1[k] < arr2[j] then --attention
            arr[i] = arr1[k]
            k = k + 1
        else
            arr[i] = arr2[j]
            j = j + 1
        end
    end
end

local mergeSort
mergeSort = function(arr, p, q)
    if p < q then
        local mid = math.floor((p + q) / 2)
        mergeSort(arr, p, mid)
        mergeSort(arr, mid + 1, q)
        mergeArr(arr, p, mid, q)
    end
end

mergeSort(arr, 1, #arr)
```

10. 堆排序

堆的排序，首先是堆的建立，然后是堆的抽离。在堆化的过程中
```lua
local arr = { 13, 20, 16, 18, 20, 12, 15, 7 }
local result = {}

local function heapify(arr, index)
    local changeIndex = index
    if arr[2 * index] and arr[index] < arr[2 * index] then
        changeIndex = 2 * index
    end

    if arr[2 * index + 1] and arr[changeIndex] < arr[2 * index + 1] then
        changeIndex = 2 * index + 1
    end
    if changeIndex ~= index then
        arr[index], arr[changeIndex] = arr[changeIndex], arr[index]
        heapify(arr, changeIndex)
    end
end

local lastParent = math.floor(#arr / 2) + 1
for i = lastParent, 1, -1 do
    heapify(arr, i)
end

local length = #arr
for i = 1, length do
    table.insert(result, arr[1])
    table.remove(arr, 1)
    heapify(arr, 1)
end

```

11. 二叉树排序

在二叉树排序之前，先来回顾一下二叉搜索树，二叉搜索树是指父节点的值大于左子树，小于右子树。
采用中序遍历即可：
```lua
local arr = { 15, 6, 18, 3, 7, 17, 20, 2, 4, 13, 9 }
local tree = {}

local function treeInsert(i)
    local node = {parent = nil,left = nil, right = nil, key = nil}
    node.key = arr[i]
    if i == 1 then
        tree.root = node
    else
        local parent = tree.root
        while parent do
            if arr[i] > parent.key then
                if  not parent.right then
                    parent.right = node
                    node.parent = parent
                    break
                else
                    parent = parent.right
                end
            elseif arr[i] <= parent.key then
                if  not parent.left then
                    parent.left = node
                    node.parent = parent
                    break
                else
                    parent = parent.left
                end
            end
        end
    end
end
for i = 1, #arr do
    treeInsert(i)
end

local prePrint
prePrint = function(node)
    if node then
        print(node.key)
        prePrint(node.left)
        prePrint(node.right)
    end
end
local midPrint
midPrint = function(node)
    if node then
        midPrint(node.left)
        print(node.key)
        midPrint(node.right)
    end
end
midPrint(tree.root)
```

前驱以及后继的判断

```lua
--1. 存在右孩子，就是最小的右孩子，
--2. 不存在右孩子，即自己的祖先或者父亲，并且这个节点的左孩子是自己或者祖先
local nextNode = function(node)
    if node.right then
        node = node.right
        while node.left do
            node = node.left
        end
        print(node.key)
        return
    end
    local p = node.parent
    while p and p.right == node do
        node = node.parent
        p = node.parent
    end
    print(p.key)
end

-- 1. 存在左边孩子，左边的第一个孩子的最后一个节点
-- 2. 父节点的左孩子
-- 3. 父节点的右孩子
local preNode = function(node)
    if node.left then
        node = node.left
        while node.right do
            node = node.right
        end
        print(node.key)
        return
    end
    local p = node.parent
    if p.left == node then
        print(p.parent.key)
    else
        print(p.key)
    end
end
```

删除： pro
```lua
--查找node的后继y，后继y没有左子树
--如果y是node的右子树，则直接拼接
--否则将y的右子树成为y父节点的左子树，再拼接
local treeDelete = function(node)
    if node.left and node.right then
        local next = nextNode(node)
        if next.parent == node then
            next.right = node.right
            next.parent = node.parent
            node.parent.right = next
        else
            next.parent.left = next.right
            next.right = node.right
            next.parent = node.parent
            node.parent.right = next
        end
    else
        node.parent.left = node.left
        node.parent.right = node.right
    end
end

treeDelete(search(6))
midPrint(tree.root)
```