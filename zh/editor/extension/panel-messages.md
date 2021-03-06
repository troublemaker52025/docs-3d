# 面板与扩展的通信

一些实用工具或者是简单的功能可以直接写在面板上，但是面板不是可靠的数据存储位置，窗口随时可能被关闭，面板也会被关闭。

最常见的例子就是某个面板被拖拽停靠到主窗口里。这时候面板会先关闭，然后在主窗口内重新打开，而面板上使用的内存里的数据如果不进行存储和备份，则会随着重启而丢失。

这时候就需要与扩展主体进行一定程度的数据交互。

在看这章节前，需要对 [消息系统](./messages.md) 有一定程度的了解。

## 定义扩展上和面板的方法

首先我们定义一份 package.json：

```json
{
    "name": "hello-world",
    "main": "./browser.js",
    "panels": {
        "default": {
            "title": "hw",
            "main": "./panel.js"
        }
    },
    "contributions": {
        "messages": {
            "upload": {
                "methods": ["saveData"]
            },
            "query": {
                "methods": ["queryData"]
            }
        }
    }
}
```

然后定义扩展的 main 文件 browser.js：

```javascript
exports.methods = {
    saveData(path, data) {
        // 收到数据后缓存起来
        this.cache[path] = data;
    },
    queryData(path) {
        const result = this.cache[path];
        delete this.cache[path];
        return result;
    },
};

exports.load = function() {};
exports.unload = function() {};
```

然后定义面板的 main 文件：

 ```javascript
 exports.ready = async () => {
    const tab = await Editor.Message.send('hello-world', 'query', 'tab');
    const subTab = await Editor.Message.send('hello-world', 'query', 'subTab');

    // 打印查询到的数据
    console.log(tab, subTab):
    // TODO 使用这两个数据初始化
 };
 exports.close() {
        // 收到数据后上传到扩展进程
        Editor.Message.send('hello-world', 'upload', 'tab', 1);
        Editor.Message.send('hello-world', 'upload', 'subTab', 0);
    },
 };
 ```

 ## 发送消息

 当我们定义好扩展和扩展里的面板后，就可以尝试触发这些消息。

 按下 ctrl(cmd) + shift + i 打开控制台。在控制台打开面板:

 ```javascript
 // default 可以省略，如果面板名字是非 default，则需要填写 'hello-world.xxx'
 Editor.Panel.open('hello-world');
 ```

打开面板后，控制台会打印出一句:

```
undefined, undefined
```

这是因为数据还没有提交。让我们关闭这个面板，然后再次打开，这时候控制台打印出了数据：

 ```
1, 0
 ```

 因为面板在关闭的时候，发送了两条消息：

```javascript
Editor.Message.send('hello-world', 'upload', 'tab', 1);
Editor.Message.send('hello-world', 'upload', 'subTab', 0);
```

通过这两条消息，Message 系统首先根据 messages 里的 upload 定义（"methods": ["saveData"]），将数据保存到扩展进程里。

而在再次打开面板的时候，通过：

```javascript
const tab = await Editor.Message.send('hello-world', 'query', 'tab');
const subTab = await Editor.Message.send('hello-world', 'query', 'subTab');
```

查询到刚刚保存的数据，并初始化界面、打印到控制台。

至此，我们完成了一次面板与扩展进程的交互。
