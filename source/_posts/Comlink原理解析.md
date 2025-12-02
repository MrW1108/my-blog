---
title: Comlink原理解析
summary: 从架构、序列化系统、Proxy 拦截、对象句柄、Promise 管理、message 协议、生命周期管理，到源码级别的流程拆解
date: 2025-12-01
cover: true
top: false
categories: 前端
tags:
  - JavaScript
  - WebWorker
  - 多线程
---

## Comlink 的核心目标：让 Worker 通信像调用本地函数一样

### Conlink 解决了什么问题
传统 Worker 通信：
```js
// main.js
worker.postMessage({ type: "ADD", a: 1, b: 2 });
worker.onmessage = (e) => console.log(e.data);

// worker.js
onmessage = (e) => {
  const { type, payload } = e.data;

  if (type === 'add') {
    const { a, b } = payload;
    const result = a + b;
    postMessage(result);
  }
};
```

缺点：
- 需要定义 message type / payload
- 数据需要手动封装
- 回调地狱
- 不能传函数
- 不能传对象方法调用
- 不支持 await workerFn()

Comlink 把这两件事自动化：

1. **通信层抽象成 RPC**
2. **远程对象抽象为本地 Proxy**

所以你调用：

```js
const api = Comlink.wrap(worker.js)
await api.add(1,2)
```

实际上等价于：

```js
worker.postMessage({
  type: "APPLY",
  path: ["add"],
  argumentList: [
    { type: "RAW", value: 1 },
    { type: "RAW", valueL: 2 },
  ],
  id: "xxx",
});
await waitForResponse(id);
```

本质是：

> Comlink = Proxy 捕获 API 调用 → 自动序列化/transferable，转为 message 发给 worker，返回 Promise → worker 执行 → 把结果打包成消息回传 → 主线程的 Promise resolve， 执行回调函数

github地址：[Comlink](https://github.com/GoogleChromeLabs/comlink)

---

## Comlink 组件

| 组件                  | 作用                                         |
|---------------------|--------------------------------------------|
| `wrap()`             | 将 worker 代理成一个 `Proxy`                 |
| `expose()`           | 在 worker 中暴露对象                          |
| `serialize()` / `deserialize()` | 转为可跨线程传输的 wire 数据格式        |
| `endpoint`           | `postMessage` + `addEventListener` 的包装  |
| object registry      | 存远程对象引用（`Proxy` 句柄）               |
| request-response map | 用 `id` 映射 `Promise`                        |

---

## Comlink 消息协议

Comlink 使用极小的协议：

| type      | 作用     |
| --------- | ------ |
| GET       | 读取属性   |
| SET       | 写属性    |
| APPLY     | 调用函数   |
| CONSTRUCT | new    |
| RELEASE   | 删除远程对象 |

一个函数调用最终会生成：

```js
{
  type: "APPLY",
  callPath: ["database", "query"],
  argumentList: [
    { "type": "RAW", "value": 18 }
  ]
  id: "req-xxxx"
}
```

Worker 执行后：

```js
{
  type: "RESULT",
  id: "req-xxxx",
  result: 123
}
```

---

## wrap() 原理 -- 利用 Proxy 拦截所有操作

wrap(worker) 返回 Proxy：

```js
const api = wrap(worker)
```
Proxy 拦截三种操作：
- get → 读取属性
- set → 设置属性
- apply → 调用函数

核心伪代码
```js
function createProxy(endPointer, path = []) {
  return new Proxy(endPointer, {
    get(_, prop){
      // 一些处理 ...
      return createProxy(endPointer, [...path, prop])
    },
    apply(_, _, args) {
      // 一些处理 ...
      return requestResponseMessage("APPLY", path, args); // Promise
    },
    set(_, prop, value){
       // 一些处理 ...
      return requestResponseMessage("SET", [...path, props], value); // Promise
    }
  })
}
```
- “读取属性”只是构建 path，不会通信
- “调用函数”才真正发消息

---

## expose()原理
Worker 端暴露对象：
```js
const exposedObject = {
  add(a, b) { return a + b }
}
Comlink.expose(exposedObject);
```

expose 会监听消息：
```js
onmessage = async (event) => {
  const { id, type, path, argumentList } = event.data;

  const target = path.reduce((o, key) => o[key], exposedObject);

  if (type === "APPLY") {
    const args = deserialize(argumentList);
    const result = await target(...args);
    endpoint.postMessage({ id, type: "RESULT", result });
  }
};
```
流程非常简单：
- 根据 path 找到被调用的函数
- 调用函数
- 把结果返回

---

## 参数 & 返回值序列化机制

Comlink 强化了结构化克隆：

| Input                      |     Output     | Notes                                                                                        |
| -------------------------- | :------------: | -------------------------------------------------------------------------------------------- |
| `[1,2,3]`                  |   `[1,2,3]`    | Full copy                                                                                    |
| `{a: 1, b: 2}`             | `{a: 1, b: 2}` | Full copy                                                                                    |
| `{a: 1, b() { return 2; }` |    `{a: 1}`    | Full copy, 忽略 functions                                                     |
| `new MyClass()`            |    `{...}`     | 仅仅包含 properties                                                                          |
| `Map`                      |     `Map`      | [`Map`][map] 是 structured cloneable                                                         |
| `Set`                      |     `Set`      | [`Set`][set] 是 structured cloneable                                                         |
| `ArrayBuffer`              | `ArrayBuffer`  | [`ArrayBuffer`][arraybuffer] 是 structured cloneable                                         |
| `Uint32Array`              | `Uint32Array`  | [`Uint32Array`][uint32array] 以及所有其他 type 的数组都是 structured cloneable         |
| `Event`                    |       ❌       |                                                                                              |
| Any DOM element            |       ❌       |                                                                                              |
| `MessagePort`              |       ❌       | 仅能 transferable, 不是 structured cloneable                                                  |
| `Request`                  |       ❌       |                                                                                              |
| `Response`                 |       ❌       |                                                                                              |
| `ReadableStream`           |       ❌       | [Streams are planned to be transferable][transferable streams], 不是 structured cloneable |

---

## Promise 实现（request-response）

wrap 调用远程方法时：

```js
const msgId = generateId();

endpoint.postMessage(message);

return new Promise(resolve => {
    pendingMap[msgId] = resolve;
});

```

主线程中维护一个 Map：

```js
pendingRequests[id] = { resolve, reject }
```

Worker 返回时：

```js
endpoint.postMessage({ id, type:"RESULT", result });
// 主线程收到：
pendingMap[id](result);
delete pendingMap[id];
```