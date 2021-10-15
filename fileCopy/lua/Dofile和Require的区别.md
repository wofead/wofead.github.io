# Dofile和Require的区别

## require和dofile区别

在加载一个.lua文件的时候，require会先在package.loaded中查找此模块是否存在，如果存在则直接返回模块，如果不存在，则加载此模块。

dofile会对读入的模块编译执行，每调用dofile一次，都会重新编译执行一次。

require它的参数只是文件名，而dofile要求参数必须带上文件名的后缀。

`foo.lua`

```lua
local class = {}
function class.add(a, b)
    return a + b
end
```

`test.lua`

```lua
print("require:")
for i = 1, 2 do
    print(require("foo"))
end
for i = 1, 2 do
    print(dofile("foo"))
end

-- require:
-- table: 0x995cfc8
-- table: 0x995cfc8
-- table: 0x995d868
-- table: 0x995dc30
```



总结： 可以看到，对于同一个foo模块，require和dofile分别调用两次的结果是不同的。require只加载了 一次。

### loadfile与dofile的区别

loadfile只编译模块，但是不执行模块，loadfile将模块编译之后当做函数返回。dofile编译并执行模块。

例：用loadfile打开foo模块，并打印结果，可以看到loadfile是只编译，不执行的。