# Lua常用的技巧集合

[toc]

## 中文字符串的裁剪

这里利用utf8函数进行字节位数的获取以及字符串的裁剪。

```lua
local name = "你的名字是jow"
print(utf8.offset(name, 2))
local subString = string.sub(name, 1, utf8.offset(name, 3) - 1)
print(subString)
```

