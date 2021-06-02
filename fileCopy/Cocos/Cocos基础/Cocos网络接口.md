# 网络接口

[toc]

# 标准网络接口

在 Cocos Creator 中，我们支持 Web 平台上最广泛使用的标准网络接口：

- **XMLHttpRequest**：用于短连接
- **WebSocket**：用于长连接

当然，在 Web 平台，浏览器原生就支持这两个接口，之所以说 Cocos Creator 支持，是因为在发布原生版本时，用户使用这两个网络接口的代码也是可以运行的。也就是遵循 Cocos 一直秉承的 “一套代码，多平台运行” 原则。

> **注意**：如果需要在原生平台使用 `WebSocket`，请确保有在 **项目 -> 项目设置 -> 模块设置** 中勾选了 **Native Socket** 模块。

## 使用方法

1. XMLHttpRequest

   简单示例：

   ```js
    let xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4 && (xhr.status >= 200 && xhr.status < 400)) {
            var response = xhr.responseText;
            console.log(response);
        }
    };
    xhr.open("GET", url, true);
    xhr.send();
   ```

   开发者可以直接使用 `new XMLHttpRequest()` 来创建一个连接对象。

   `XMLHttpRequest` 的标准文档请参考 [MDN 中文文档](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)。

2. WebSocket

   简单示例：

   ```js
    let ws = new WebSocket("ws://echo.websocket.org");
    ws.onopen = function (event) {
        console.log("Send Text WS was opened.");
    };
    ws.onmessage = function (event) {
        console.log("response text msg: " + event.data);
    };
    ws.onerror = function (event) {
        console.log("Send Text fired an error");
    };
    ws.onclose = function (event) {
        console.log("WebSocket instance closed.");
    };
   
    setTimeout(function () {
        if (ws.readyState === WebSocket.OPEN) {
            ws.send("Hello WebSocket, I'm a text message.");
        }
        else {
            console.log("WebSocket instance wasn't ready...");
        }
    }, 3);
   ```

   `WebSocket` 的标准文档请参考文档 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)。