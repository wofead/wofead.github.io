# 字符串转换为整形

> 请你来实现一个 `atoi` 函数，使其能将字符串转换成整数。
>
> 首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。接下来的转化规则如下：
>
> - 如果第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字字符组合起来，形成一个有符号整数。
> - 假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成一个整数。
> - 该字符串在有效的整数部分之后也可能会存在多余的字符，那么这些字符可以被忽略，它们对函数不应该造成影响。
>
> 假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换，即无法进行有效转换。
>
> 在任何情况下，若函数不能进行有效的转换时，请返回 0 。
>
> **注意：**假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为[−2^31, 2^31 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 最大值或者最小值。
>
> **提示：**
>
> * `0 <= s.length <= 200`
> * `s` 由英文字母（大写和小写）、数字、`' '`、`'+'`、`'-'` 和 `'.'` 组成

## 解题思路
- 判断字符是否位于数字字符范围内。

- 如果不含有特殊字符了，或者遇到dot直接返回值。

  

## 注意：

这种方法很危险，`temp = rev * 10 + pop;`会导致溢出。

1. 如果`temp = rev * 10 + pop;`导致溢出，那么一定有 `rev > int.MaxValue / 10`.
2. 如果 `rev > int.MaxValue / 10`.那么`temp = rev * 10 + pop;`一定溢出。
3. 如果`rev == int.MaxValue / 10`,那么只要`pop>7`，`temp = rev * 10 + pop;`一定溢出。
4. 负数类似，只不过7变成-8.


## 解：

```c#
using System.Collections.Generic;

namespace CodeExercise
{
    public class MyAtoi
    {
        public static int Solve(string s)
        {
            int len = s.Length;
            List<char> canMeet = new List<char>();
            bool isNeg = false;
            canMeet.Add(' ');
            canMeet.Add('+');
            canMeet.Add('-');
            int ret = 0;
            for (int i = 0; i < len; i++)
            {
                if (s[i] <= '9' && s[i] >= '0')
                {
                    if (canMeet.Count > 1)
                    {
                        canMeet.Remove(' ');
                        canMeet.Remove('+');
                        canMeet.Remove('-');
                    }
                    int pop = s[i] - '0';
                    if (isNeg)
                    {
                        pop = -pop;
                    }
                    if (ret > int.MaxValue / 10 || (ret == int.MaxValue / 10 && pop > 7)) return int.MaxValue;
                    if (ret < int.MinValue / 10 || (ret == int.MinValue / 10 && pop < -8)) return int.MinValue;
                    ret = ret * 10 + pop;
                    continue;
                }
                if (!canMeet.Contains(s[i]) || s[i] == '.')
                {
                    return ret;
                }
                if (s[i] == '+')
                {
                    canMeet.Remove(' ');
                    canMeet.Remove('+');
                    canMeet.Remove('-');
                }
                if (s[i] == '-')
                {
                    canMeet.Remove(' ');
                    canMeet.Remove('+');
                    canMeet.Remove('-');
                    isNeg = true;
                }
            }
            return ret;
        }
    }
}


```

## 其他解法

这里用位运算最为简单，使用DFA有限状态机比较明了。

```c++
class Automaton {
    string state = "start";
    unordered_map<string, vector<string>> table = {
        {"start", {"start", "signed", "in_number", "end"}},
        {"signed", {"end", "end", "in_number", "end"}},
        {"in_number", {"end", "end", "in_number", "end"}},
        {"end", {"end", "end", "end", "end"}}
    };

    int get_col(char c) {
        if (isspace(c)) return 0;
        if (c == '+' or c == '-') return 1;
        if (isdigit(c)) return 2;
        return 3;
    }
public:
    int sign = 1;
    long long ans = 0;

    void get(char c) {
        state = table[state][get_col(c)];
        if (state == "in_number") {
            ans = ans * 10 + c - '0';
            ans = sign == 1 ? min(ans, (long long)INT_MAX) : min(ans, -(long long)INT_MIN);
        }
        else if (state == "signed")
            sign = c == '+' ? 1 : -1;
    }
};

class Solution {
public:
    int myAtoi(string str) {
        Automaton automaton;
        for (char c : str)
            automaton.get(c);
        return automaton.sign * automaton.ans;
    }
};
```

```c++
class Solution {
public:
    int myAtoi(string str) {
        long long int ans=0;
        int flag=0;//用每一位分别存储状态信息，初始位0000（只用到四位）
        for(int i=0;i<str.size();i++){
            if((flag&1) == 0 && str[i]==' ')
                continue;
            flag|=1;//此时位0001，代表空字符判断结束
            if((flag&14)==0 && (str[i]=='+' || str[i]=='-')){//14=1110，3个1分别代表出现数字、符号位为“+”和符号位为“-”。显然需要这三位都为0才可以进行符号位判断
                flag|=(str[i]=='-'?2:4);//'+'则和0100取或，加上4'-'则和0010取或
                continue;
            }
            if(str[i]>='0' && str[i]<='9'){
                flag|=8;
                ans=10*ans+str[i]-'0';//'0'用来将ascll码转换成数字（利用其排序规则）
                if(ans>(long long)INT_MAX)//每加一位数字都进行溢出判断，防止数字过大
                    return (flag&2)==2?INT_MIN:INT_MAX;
                continue;
            }
            break;//判断首次出现数字后是否出现非数字字符，出现则结束循环
        }
        return (flag&2)==2?-ans:ans;
    }
};
```

