# 迁移到 Express 5

## 概述

Express 5.0 目前仍处于 alpha 版本阶段，但是下面是即将发布的版本中的更改预览，以及如何将您的 Express 4 应用迁移到 Express 5。

Express 5 与 Express 4 相比没有太大的区别：API 的变化并不像从 3.0 到 4.0 那样显著。尽管基本的 API 保持不变，但仍然存在着一些破坏性的变更；换句话说，如果您将应用程序更新到使用 Express 5，现有的 Express 4 程序可能无法正常工作。

要安装最新的 alpha 版本并预览 Express 5，请在您的应用根目录中输入以下命令：

```console
$ npm install express@>=5.0.0-alpha.8 --save
```

然后，您可以运行自动化测试，查看哪些测试失败，并根据下面的更新修复问题。在解决测试失败后，运行您的应用程序，查看发生了哪些错误。您会立即发现是否使用了任何不受支持的方法或属性。

## Express 5 中的变化

以下是将影响到您作为 Express 用户的变化列表（截至 alpha 2 发布）。可以查看[拉取请求](https://github.com/expressjs/express/pull/2237)以获取所有计划特性的列表。

**已移除的方法和属性**

- [app.del()](https://www.expressjs.com.cn/guide/migrating-5.html#app.del)
- [app.param(fn)](https://www.expressjs.com.cn/guide/migrating-5.html#app.param)
- [复数形式的方法名](https://www.expressjs.com.cn/guide/migrating-5.html#plural)
- [app.param(name, fn) 中 name 参数中的前导冒号](https://www.expressjs.com.cn/guide/migrating-5.html#leading)
- [req.param(name)](https://www.expressjs.com.cn/guide/migrating-5.html#req.param)
- [res.json(obj, status)](https://www.expressjs.com.cn/guide/migrating-5.html#res.json)
- [res.jsonp(obj, status)](https://www.expressjs.com.cn/guide/migrating-5.html#res.jsonp)
- [res.send(body, status)](https://www.expressjs.com.cn/guide/migrating-5.html#res.send.body)
- [res.send(status)](https://www.expressjs.com.cn/guide/migrating-5.html#res.send.status)
- [res.sendfile()](https://www.expressjs.com.cn/guide/migrating-5.html#res.sendfile)

**已更改的部分**

- [app.router](https://www.expressjs.com.cn/guide/migrating-5.html#app.router)
- [req.host](https://www.expressjs.com.cn/guide/migrating-5.html#req.host)
- [req.query](https://www.expressjs.com.cn/guide/migrating-5.html#req.query)

**改进**

- [res.render()](https://www.expressjs.com.cn/guide/migrating-5.html#res.render)

### 已移除的方法和属性

如果您在应用程序中使用了以下任何方法或属性，它将会崩溃。因此，在更新到版本 5 后，您需要更改您的应用程序。

#### app.del()

Express 5 不再支持 `app.del()` 函数。如果使用此函数，将会抛出错误。要注册 HTTP DELETE 路由，请改用 `app.delete()` 函数。

最初，`del` 被用于代替 `delete`，因为在 JavaScript 中 `delete` 是一个保留关键字。然而，从 ECMAScript 6 开始，`delete` 和其他保留关键字可以合法地用作属性名。

#### app.param(fn)

`app.param(fn)` 签名用于修改 `app.param(name, fn)` 函数的行为。自 v4.11.0 以来，它已被弃用，并且 Express 5 不再支持它。

#### 复数形式的方法名

以下方法名已经变为复数形式。在 Express 4 中，使用旧方法会导致警告。Express 5 不再支持这些方法：

- `req.acceptsCharset()` 被替换为 `req.acceptsCharsets()`。
- `req.acceptsEncoding()` 被替换为 `req.acceptsEncodings()`。
- `req.acceptsLanguage()` 被替换为 `req.acceptsLanguages()`。

#### 在 app.param(name, fn) 的 name 参数中的前导冒号 (:)

在 `app.param(name, fn)` 函数的 name 参数中的前导冒号字符 (:) 是 Express 3 的遗留物，为了向后兼容，Express 4 支持它并显示了弃用通知。Express 5 将会忽略它，并使用不带冒号前缀的 name 参数。

如果您遵循 Express 4 的 [app.param](https://www.expressjs.com.cn/en/4x/api.html

#app.param) 文档，这不应该影响您的代码，因为文档中没有提及前导冒号。

#### req.param(name)

已移除此种可能会令人困惑且危险的检索表单数据的方法。现在您需要在 `req.params`、`req.body` 或 `req.query` 对象中专门查找提交的参数名。

#### res.json(obj, status)

Express 5 不再支持 `res.json(obj, status)` 的签名。取而代之的是，首先设置状态，然后将其链接到 `res.json()` 方法，例如：`res.status(status).json(obj)`。

#### res.jsonp(obj, status)

Express 5 不再支持 `res.jsonp(obj, status)` 的签名。取而代之的是，首先设置状态，然后将其链接到 `res.jsonp()` 方法，例如：`res.status(status).jsonp(obj)`。

#### res.send(body, status)

Express 5 不再支持 `res.send(obj, status)` 的签名。取而代之的是，首先设置状态，然后将其链接到 `res.send()` 方法，例如：`res.status(status).send(obj)`。

#### res.send(status)

Express 5 不再支持 `res.send(*status*)` 的签名，其中 *`status`* 是一个数字。取而代之的是，使用 `res.sendStatus(statusCode)` 函数，它设置 HTTP 响应头状态码并发送代码的文本版本，例如：“Not Found”、“Internal Server Error” 等等。如果您需要使用 `res.send()` 函数发送一个数字，请将数字引用起来，将其转换为字符串，以便 Express 不将其解释为尝试使用不受支持的旧签名。

#### res.sendfile()

在 Express 5 中，`res.sendfile()` 函数已被驼峰命名版本 `res.sendFile()` 取代。

### 已更改的部分

#### app.router

`app.router` 对象在 Express 4 中被移除，但在 Express 5 中已经重新引入。在新版本中，此对象仅仅是对基础 Express 路由器的引用，不像在 Express 3 中，应用程序必须显式地加载它。

#### req.host

在 Express 4 中，`req.host` 函数会错误地删除掉端口号，如果存在的话。在 Express 5 中，端口号会被保留。

#### req.query

在 Express 4.7 和 Express 5 及以后版本中，查询解析选项可以接受 `false` 来禁用查询字符串解析，当您希望使用自己的函数进行查询字符串解析逻辑时可以这样做。

### 改进

#### res.render()

这个方法现在对于所有视图引擎都强制执行异步行为，避免了由视图引擎具有同步实现引起的 bug，因为这些实现违反了推荐的接口。