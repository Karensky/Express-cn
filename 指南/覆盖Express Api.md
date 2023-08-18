# 覆盖 Express API

Express API 包含了请求和响应对象上的各种方法和属性。这些是通过原型继承的。Express API 有两个扩展点：

1. 全局原型在 `express.request` 和 `express.response` 上。
2. 应用程序特定的原型在 `app.request` 和 `app.response` 上。

更改全局原型将影响同一进程中加载的所有 Express 应用程序。如果需要，可以通过在创建新应用程序后仅更改应用程序特定的原型来进行应用程序特定的更改。

## 方法

您可以通过分配自定义函数来覆盖现有方法的签名和行为。

以下是覆盖 [res.sendStatus](https://www.expressjs.com.cn/4x/api.html#res.sendStatus) 行为的示例。

```javascript
app.response.sendStatus = function (statusCode, type, message) {
  // 代码故意保持简单，仅用于演示目的
  return this.contentType(type)
    .status(statusCode)
    .send(message)
}
```

上述实现完全改变了 `res.sendStatus` 的原始签名。它现在接受状态码、编码类型和要发送到客户端的消息。

现在可以这样使用覆盖的方法：

```javascript
res.sendStatus(404, 'application/json', '{"error":"resource not found"}')
```

## 属性

Express API 中的属性要么是：

1. 分配的属性（例如：`req.baseUrl`、`req.originalUrl`）
2. 定义为 getter（例如：`req.secure`、`req.ip`）

由于第一类属性是在当前请求-响应周期的上下文中动态分配到 `request` 和 `response` 对象上的，它们的行为无法被覆盖。

第二类属性可以通过 Express API 扩展 API 进行覆盖。

以下代码重新定义了如何派生 `req.ip` 的值。现在，它只是返回 `Client-IP` 请求头的值。

```javascript
Object.defineProperty(app.request, 'ip', {
  configurable: true,
  enumerable: true,
  get: function () { return this.get('Client-IP') }
})
```