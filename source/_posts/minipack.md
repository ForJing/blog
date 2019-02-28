---
title: webpack打包原理分析
date: 2019-02-28 15:52:49
tags:
  - 前端
  - webpack
---

webpack 看起来很神奇， 其实它的打包原理就是读取文件， 分析文件，然后对代码做一些处理， 然后输出成新的文件。

首先我们建立如下的目录结构

![](./webpack.png)

后面我们只会专注于 `mypack.js` 文件

首先， 我们要能读取文件内容

```js
const fs = require("fs");

function createAsset(filename) {
  const content = fs.readFileSync(filename, "utf-8");
  console.log(content);
}

createAsset("./example/entry.js");
```

当我们执行 `$ node mypack.js`
我们会看到如下输入

```
import message from "./message.js";

console.log(message);
```

现在仅仅是输入了 `entry` 文件的内容， 接下来， 让我们来处理这个程序

首先我们运行`$ npm install @babel/parser @babel/traverse`安装两个包.
其中`parser`是把程序转换为`AST`, `traverse`是用来遍历`ast`的

关于 AST(抽象语法树)可以自行搜索, 也可以看看 [云谦大大的 AST 入门](https://www.bilibili.com/video/av37835266?from=search&seid=4932665822458895985)

在 [https://astexplorer.net/](https://astexplorer.net/) 这个网站可以在线运行 ast， 其中 compiler 为 babylon7 就是 babel-parser 内部用的编译器.

编写代码

```js
const fs = require("fs");
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;

let ID = 0;

function createAsset(filename) {
  const content = fs.readFileSync(filename, "utf-8");

  const ast = parser.parse(content, { sourceType: "module" });

  const dependencies = [];

  traverse(ast, {
    ImportDeclaration({ node }) {
      dependencies.push(node.source.value);
    }
  });

  const id = ID++;

  return {
    id,
    filename,
    dependencies
  };
}

console.log(createAsset("./example/entry.js"));
```

ast 如下， node 便是你所遍历的类型的节点， node => source->value 既是引入文件的路径
![](./ast2.png)

在上面代码中， 我们声明一个全局的 ID 来储存模块 ID， `createAsset`函数找出文件的所有依赖
现在我们再执行函数， 会得到如下结果

```json
{
  "id": 0,
  "filename": "./example/entry.js",
  "dependencies": ["./message.js"]
}
```

接下来，我们写一个`createGraph`函数来循环创建一个依赖图

```js
function createGraph(entry) {
  const mainAsset = createAsset(entry);

  const queue = [mainAsset];

  for (const asset of queue) {
    asset.mapping = {};
    // 此模块的目录
    const dirname = path.dirname(asset.filename);

    asset.dependencies.forEach(relativePath => {
      const absolutePath = path.join(dirname, relativePath);
      const child = createAsset(absolutePath);
      asset.mapping[relativePath] = child.id;
      queue.push(child);
    });
  }
  return queue;
}
```

上面用一个队列来循环构建依赖图， 其中要注意路径的处理
现在执行`console.log(createGraph("./example/entry.js"));`
会看到一下输出

```json
[
  {
    "id": 0,
    "filename": "./example/entry.js",
    "dependencies": ["./message.js"],
    "mapping": { "./message.js": 1 }
  },
  {
    "id": 1,
    "filename": "example/message.js",
    "dependencies": ["./name.js"],
    "mapping": { "./name.js": 2 }
  },
  { "id": 2, "filename": "example/name.js", "dependencies": [], "mapping": {} }
]
```

可以看到 `entry` 依赖 `message`, `message` 又依赖 `name`

未完待续
