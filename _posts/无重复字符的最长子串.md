# 无重复字符的最长子串

> 给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。
>
> **提示：**
>
> - `0 <= s.length <= 5 * 104`
> - `s` 由英文字母、数字、符号和空格组成

## **解题思路：**

1. 首先想到使用字典来存储已经遍历过的字符
2. 发现已经存储过的字符进行更新记录
3. 使用一个前置的探针进行记录上次重复的字符位置
4. 算出探针和重复字符位置之间的距离来更新大小

## **遇到的问题：**

### 1. 遇到重复的字符就清理字典

这会导致一个问题，如果重复的两个字符不是紧挨者的话，你把前面的字符的记录也全部清理了，就不能找到最大字符串了。

> `adfghjfsegbte`，在这个例子中，第二个f之前的都被清理掉了，那`ghjf`这个字符串就丢失了

所以我们不能清理掉字典，只能更新字典。

### 2. 字典的更新

怎么更新字典呢？我们只需要将重复的那个字符的值更新为后面一个字符的位置就可以了。

### 3.字符串长度的获取与重复字符的更新时机

并不是每次判断到有重复字符我们就要更新，还要比较一下，重复的那个字符的值是否在前置探针之前，之前就不需要更新了，只需要更新字典中的值就可以了。

### 4.最后一个字符的问题

如果最后一个字符是重复的，就不需要特殊判断了，如果不是重复的，就需要特殊判断一下。

## 解！！！！！！！

```c#
Dictionary<char, int> cacheString = new Dictionary<char, int>();
int front = 0;
int result = 0;
bool lastRepeat = false;
for (int i = 0; i < s.Length; i++)
{
    if (cacheString.ContainsKey(s[i]) && front < cacheString[s[i]] + 1)
    {
        if (i - front > result)
        {
            result = i - front;
        }
        front = cacheString[s[i]] + 1;
        cacheString[s[i]] = i;
        if (i == s.Length - 1)
        {
            lastRepeat = true;
        }
    }
    else
    {
        cacheString[s[i]] = i;
    }
}
if (!lastRepeat && s.Length - front > result)
{
    result = s.Length - front;
}
return result;
```

> 执行用时: **104 ms**
>
> 内存消耗: **25.6 MB**

## 官方解：

使用的是滑动窗口：

我们不妨以示例一中的字符串`abcabcbb`为例，找出 从每一个字符开始的，不包含重复字符的最长子串，那么其中最长的那个字符串即为答案。对于示例一中的字符串，我们列举出这些结果，其中括号中表示选中的字符以及最长的字符串：

- 以`(a)bcabcbb` 开始的最长字符串为`(abc)abcbb`；
- 以`a(b)cabcbb` 开始的最长字符串为`a(bca)bcbb`；
- 以`ab(c)abcbb` 开始的最长字符串为`ab(cab)cbb`；
- 以`abc(a)bcbb` 开始的最长字符串为`abc(abc)bb`；
- 以`abca(b)cbb` 开始的最长字符串为`abca(bc)bb`；
- 以`abcab(c)b` 开始的最长字符串为`abcab(cb)b`；
- 以`abcabc(b)b` 开始的最长字符串为`abcabc(b)b`；
- 以`abcabcb(b)` 开始的最长字符串为`abcabcb(b)`；

### 	

```c#
public static int OfficialSolve(string s)
{
    int result = 0;
    int sLength = s.Length;
    int rk = -1;
    HashSet<char> occ = new HashSet<char>();
    for (int i = 0; i < sLength; i++)
    {
        if (i != 0)
        {
            occ.Remove(s[i - 1]);
        }
        while (rk + 1 < sLength && !occ.Contains(s[rk + 1]))
        {
            occ.Add(s[rk + 1]);
            ++rk;
        }
        result = Math.Max(result, rk - i + 1);
    }
    return result;
}
```

> 执行用时: **96 ms**
>
> 内存消耗: **25.9 MB**