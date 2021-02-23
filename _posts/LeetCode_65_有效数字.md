# 有效数字

> 有效数字（按顺序）可以分成以下几个部分：
>
> 1. 一个 小数 或者 整数
> 2. （可选）一个 'e' 或 'E' ，后面跟着一个 整数
>
> 小数（按顺序）可以分成以下几个部分：
>
> 1. （可选）一个符号字符（'+' 或 '-'）
> 2. 下述格式之一：
>    1. 至少一位数字，后面跟着一个点 '.'
>    2. 至少一位数字，后面跟着一个点 '.' ，后面再跟着至少一位数字
>    3. 一个点 '.' ，后面跟着至少一位数字
>
> 整数（按顺序）可以分成以下几个部分：
>
> 1. （可选）一个符号字符（'+' 或 '-'）
> 2. 至少一位数字

## 结题思路

1. 状态机，然后不断更新状态机
2. 我使用一个List进行状态机的维护
3. 其实应该使用状态机的


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class StringIsNumber
    {
        //第一个字符可以是sign,小数点，整数
        string validStr = "+-.eE0123456789";
        bool hasInt = false;
        public bool Solve(string s)
        {
            List<char> state = new List<char>();
            state.Add('s');//符号
            state.Add('d');//点
            //state.Add('e');//e
            state.Add('i');//整数
            List<char> uniqueChar = new List<char>();
            int length = s.Length;
            for (int i = 0; i < length; i++)
            {
                if (ChangeState(s[i], state, uniqueChar))
                {
                    continue;
                }
                return false;
            }
            if (!hasInt)
            {
                return false;
            }
            return true;
        }

        public bool ChangeState(char c, List<char> state, List<char> uniqueChar)
        {
            int index = validStr.IndexOf(c);
            bool justAdd = false;
            if (index == -1) return false;
            else if (index <= 1)
            {
                if (state.Contains('s'))
                {
                    state.Remove('s');
                }
                else
                {
                    return false;
                }
            }
            else if (index <= 2)
            {
                if (state.Contains('d'))
                {
                    state.Remove('d');
                }
                else
                {
                    return false;
                }
            }
            else if (index <= 4)
            {
                if (state.Contains('e'))
                {
                    state.Add('s');
                    justAdd = true;
                    state.Remove('e');
                    state.Remove('d');
                    hasInt = false;
                }
                else
                {
                    return false;
                }
            }
            else
            {
                hasInt = true;
                if (!uniqueChar.Contains('e'))
                {
                    uniqueChar.Add('e');
                    state.Add('e');
                }
            }
            if (state.Contains('s') && !justAdd) state.Remove('s');
            return true;
        }

    }
}

```



```c++
class Solution {
public:
    enum State {
        STATE_INITIAL,
        STATE_INT_SIGN,
        STATE_INTEGER,
        STATE_POINT,
        STATE_POINT_WITHOUT_INT,
        STATE_FRACTION,
        STATE_EXP,
        STATE_EXP_SIGN,
        STATE_EXP_NUMBER,
        STATE_END,
    };

    enum CharType {
        CHAR_NUMBER,
        CHAR_EXP,
        CHAR_POINT,
        CHAR_SIGN,
        CHAR_ILLEGAL,
    };

    CharType toCharType(char ch) {
        if (ch >= '0' && ch <= '9') {
            return CHAR_NUMBER;
        } else if (ch == 'e' || ch == 'E') {
            return CHAR_EXP;
        } else if (ch == '.') {
            return CHAR_POINT;
        } else if (ch == '+' || ch == '-') {
            return CHAR_SIGN;
        } else {
            return CHAR_ILLEGAL;
        }
    }

    bool isNumber(string s) {
        unordered_map<State, unordered_map<CharType, State>> transfer{
            {
                STATE_INITIAL, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_POINT, STATE_POINT_WITHOUT_INT},
                    {CHAR_SIGN, STATE_INT_SIGN},
                }
            }, {
                STATE_INT_SIGN, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_POINT, STATE_POINT_WITHOUT_INT},
                }
            }, {
                STATE_INTEGER, {
                    {CHAR_NUMBER, STATE_INTEGER},
                    {CHAR_EXP, STATE_EXP},
                    {CHAR_POINT, STATE_POINT},
                }
            }, {
                STATE_POINT, {
                    {CHAR_NUMBER, STATE_FRACTION},
                    {CHAR_EXP, STATE_EXP},
                }
            }, {
                STATE_POINT_WITHOUT_INT, {
                    {CHAR_NUMBER, STATE_FRACTION},
                }
            }, {
                STATE_FRACTION,
                {
                    {CHAR_NUMBER, STATE_FRACTION},
                    {CHAR_EXP, STATE_EXP},
                }
            }, {
                STATE_EXP,
                {
                    {CHAR_NUMBER, STATE_EXP_NUMBER},
                    {CHAR_SIGN, STATE_EXP_SIGN},
                }
            }, {
                STATE_EXP_SIGN, {
                    {CHAR_NUMBER, STATE_EXP_NUMBER},
                }
            }, {
                STATE_EXP_NUMBER, {
                    {CHAR_NUMBER, STATE_EXP_NUMBER},
                }
            }
        };

        int len = s.length();
        State st = STATE_INITIAL;

        for (int i = 0; i < len; i++) {
            CharType typ = toCharType(s[i]);
            if (transfer[st].find(typ) == transfer[st].end()) {
                return false;
            } else {
                st = transfer[st][typ];
            }
        }
        return st == STATE_INTEGER || st == STATE_POINT || st == STATE_FRACTION || st == STATE_EXP_NUMBER || st == STATE_END;
    }
};
```

