#### 最长公共前缀

> 编写一个函数来查找字符串数组中的最长公共前缀。
>
> 如果不存在公共前缀，返回空字符串 `""`。
>
> - `0 <= strs.length <= 200`
> - `0 <= strs[i].length <= 200`
> - `strs[i]` 仅由小写英文字母组成
>

## 解：

```c#
public class Solution {
    public string LongestCommonPrefix(string[] strs) {
        if (strs.Length == 0)
        {
            return "";
        }
        string maxStr = strs[0];
        for (int i = 1; i < strs.Length; i++)
        {
            string theStr = strs[i];
            int j = 0;
            for (; j < maxStr.Length && j < theStr.Length; j++)
            {
                if (maxStr[j].Equals(theStr[j]))
                {
                    continue;
                }
                else
                {
                    break;
                }
            }
            if (j == 0)
            {
                return "";
            }
            maxStr = maxStr.Substring(0, j);
        }
        return maxStr;
    }
}


```
