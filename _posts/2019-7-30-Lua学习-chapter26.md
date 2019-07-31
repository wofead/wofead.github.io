---
layout:     post
title:      Lua 学习 chapter26
subtitle:   chapter26
date:       2019-7-31
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 使用协程实现多线程

> 只有疯狂过，你才知道自己究竟能不能成功。

## 使用协程实现多线程
我们希望通过HTTP下载多个远程文件。
```lua
local socket = require("socket")

function download(host, file)
    local c = assert(socket.connect(host, 80))
    local count = 0
    local requestURL = string.format("GET %s HTTP/1.0\r\nhost: %s\r\n\r\n", file, host)
    c:send(requestURL)
    while true do
        local s, status = receive(c)
        count = count + #s
        if status == "closed" then
            break
        end
    end
    c:close()
    print(file, count)
end

function receive(connection)
    connection:settimeout(0)
    local s, status, partial = connection:receive(2 ^ 10)
    if status == "timeout" then
        coroutine.yield(connection)
    end
    return s or partial, status
end

tasks = {} --所有活跃任务列表
function get(host, file)
    local co = coroutine.wrap(function()
        download(host, file)
    end)
    table.insert(tasks, co)
end

function dispatch()
    local i = 1
    while true do
        if tasks[i] == nil then
            if tasks[1] == nil then
                break
            end
            i = 1
        end
        local res = tasks[i]()
        if not res then
            table.remove(tasks, i)
        else
            i = i + 1
        end
    end
end

get("www.lua.org", "/ftp/lua-5.3.2.tar.gz")
get("www.lua.org", "/ftp/lua-5.3.1.tar.gz")
get("www.lua.org", "/ftp/lua-5.3.0.tar.gz")
get("www.lua.org", "/ftp/lua-5.2.4.tar.gz")
dispatch()
```


