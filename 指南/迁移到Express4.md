# 迁移到 Express 4

## 概述

Express 4 与 Express 3 相比是一个重大的变化。这意味着如果在依赖项中更新 Express 版本，现有的 Express 3 应用程序将**无法**正常工作。

本文涵盖了以下内容：

- [Express 4 中的变更](https://www.expressjs.com.cn/guide/migrating-4.html#changes)。
- [将 Express 3 应用迁移到 Express 4 的示例](https://www.expressjs.com.cn/guide/migrating-4.html#example-migration)。
- [升级到 Express 4 应用生成器](https://www.expressjs.com.cn/guide/migrating-4.html#app-gen)。

## Express 4 中的变更

Express 4 中有几个重大变更：

- [Express 核心和中间件系统的变更](https://www.expressjs.com.cn/guide/migrating-4.html#core-changes)。删除了对 Connect 的依赖以及内置中间件，因此现在必须自己添加中间件。
- [路由系统的变更](https://www.expressjs.com.cn/guide/migrating-4.html#routing)。
- [其他各种变更](https://www.expressjs.com.cn/guide/migrating-4.html#other-changes)。

另请参阅：

- [4.x 中的新特性](https://github.com/expressjs/express/wiki/New-features-in-4.x)。
- [从 3.x 迁移到 4.x](https://github.com/expressjs/express/wiki/Migrating-from-3.x-to-4.x)。

### Express 核心和中间件系统的变更

Express 4 不再依赖于 Connect，并且从其核心中删除了所有内置中间件，仅保留 `express.static` 函数。这意味着 Express 现在是一个独立的路由和中间件 Web 框架，Express 的版本和发布不会受到中间件更新的影响。

没有内置中间件，必须明确添加运行应用所需的所有中间件。只需按照以下步骤操作：

1. 安装模块：`npm install --save <module-name>`
2. 在应用中引入模块：`require('module-name')`
3. 根据模块的文档使用：`app.use( ... )`

以下表格列出了 Express 3 中的中间件及其在 Express 4 中的对应物。

| Express 3                | Express 4                                                    |
| ------------------------ | ------------------------------------------------------------ |
| `express.bodyParser`     | [body-parser](https://github.com/expressjs/body-parser) + [multer](https://github.com/expressjs/multer) |
| `express.compress`       | [compression](https://github.com/expressjs/compression)      |
| `express.cookieSession`  | [cookie-session](https://github.com/expressjs/cookie-session) |
| `express.cookieParser`   | [cookie-parser](https://github.com/expressjs/cookie-parser)  |
| `express.logger`         | [morgan](https://github.com/expressjs/morgan)                |
| `express.session`        | [express-session](https://github.com/expressjs/session)      |
| `express.favicon`        | [serve-favicon](https://github.com/expressjs/serve-favicon)  |
| `express.responseTime`   | [response-time](https://github.com/expressjs/response-time)  |
| `express.errorHandler`   | [errorhandler](https://github.com/expressjs/errorhandler)    |
| `express.methodOverride` | [method-override](https://github.com/expressjs/method-override) |
| `express.timeout`        | [connect-timeout](https://github.com/expressjs/timeout)      |
| `express.vhost`          | [vhost](https://github.com/expressjs/vhost)                  |
| `express.csrf`           | [csurf](https://github.com/expressjs/csurf)                  |
| `express.directory`      | [serve-index](https://github.com/expressjs/serve-index)      |
| `express.static`         | [serve-static](https://github.com/expressjs/serve-static)    |

以下是 [Express 4 中间件的完整列表](https://github.com/senchalabs/connect#middleware)。

在大多数情况下，您可以简单地用 Express 4 的对应中间件替换旧版本的中间件。有关详情，请参阅 GitHub 中的模块文档。

#### `app.use` 接受参数

在版本 4 中，您可以使用变量参数来定义加载中间件函数的路径，然后从路由处理程序中读取参数的值。例如：

```javascript
app.use('/book/:id', function (req, res, next) {
  console.log('ID:', req.params.id)
  next()
})
```

### 路由系统

现在应用程序隐式加载路由中间件，因此您不再需要担心中间件的加载顺序与 `router` 中间件的关系。

您定义路由的方式保持不变，但是路由系统引入了两个新功能来帮助组织路由：

- 新的方法 `app.route()` 用于为路由路径创建可链接的路由处理程序。
- 新的类 `express.Router` 用于创建可模块化挂载的路由处理程序。

#### `app.route()` 方法

新的 `app.route()` 方法使您可以为路由路径创建可链接的路由处理程序。由于路径在单个位置指定，创建模块化路由非常有帮助，减少冗余和拼写错误。有关路由的更多信息，请参阅 [`Router()` 文档](https://www.expressjs.com.cn/en/4x/api.html#router)。

以下是使用 `app.route()` 函数定义的链式路由处理程序示例。

```javascript
app.route('/book')
  .get(function (req, res) {
    res.send('Get a random book')
  })
  .post(function (req, res) {
    res.send('Add a book')
  })
  .put(function (req, res) {
    res.send('Update the book')
  })
```

#### `express.Router` 类

另一个有助于组织路

由的特性是新的 `express.Router` 类，您可以使用它来创建可模块化挂载的路由处理程序。`Router` 实例是一个完整的中间件和路由系统；因此，它经常被称为“迷你应用程序”。

以下示例创建了一个名为 `birds.js` 的路由模块，在其中加载了中间件、定义了一些路由，并将其挂载到主应用程序的路径上。

例如，在应用程序目录中创建一个名为 `birds.js` 的路由文件，内容如下：

```javascript
var express = require('express')
var router = express.Router()

// 特定于此路由的中间件
router.use(function timeLog (req, res, next) {
  console.log('Time: ', Date.now())
  next()
})
// 定义主页路由
router.get('/', function (req, res) {
  res.send('Birds home page')
})
// 定义关于路由
router.get('/about', function (req, res) {
  res.send('About birds')
})

module.exports = router
```

然后，在应用程序中加载路由模块：

```javascript
var birds = require('./birds')

// ...

app.use('/birds', birds)
```

现在，应用程序将能够处理到 `/birds` 和 `/birds/about` 路径的请求，并且将调用特定于路由的 `timeLog` 中间件。

### 其他变更

以下表格列出了 Express 4 中的其他一些小但重要的变更：

| 对象                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| Node.js                            | Express 4 需要 Node.js 0.10.x 或更高版本，并停止支持 Node.js 0.8.x。 |
| `http.createServer()`              | 除非需要直接使用它（socket.io/SPDY/HTTPS），否则不再需要 `http` 模块。可以使用 `app.listen()` 函数启动应用程序。 |
| `app.configure()`                  | 移除了 `app.configure()` 函数。使用 `process.env.NODE_ENV` 或 `app.get('env')` 函数检测环境并相应地配置应用程序。 |
| `json spaces`                      | 在 Express 4 中，默认禁用了 `json spaces` 应用程序属性。     |
| `req.accepted()`                   | 使用 `req.accepts()`、`req.acceptsEncodings()`、`req.acceptsCharsets()` 和 `req.acceptsLanguages()`。 |
| `res.location()`                   | 不再解析相对 URL。                                           |
| `req.params`                       | 从数组变为对象。                                             |
| `res.locals`                       | 从函数变为对象。                                             |
| `res.headerSent`                   | 改为 `res.headersSent`。                                     |
| `app.route`                        | 现在作为 `app.mountpath` 可用。                              |
| `res.on('header')`                 | 已移除。                                                     |
| `res.charset`                      | 已移除。                                                     |
| `res.setHeader('Set-Cookie', val)` | 功能现在仅限于设置基本的 cookie 值。使用 `res.cookie()` 获得更多功能。 |

## 示例应用程序迁移

下面是将 Express 3 应用程序迁移到 Express 4 的示例。涉及的文件有 `app.js` 和 `package.json`。

### 版本 3 应用程序

#### `app.js`

考虑一个具有以下 `app.js` 文件的 Express v.3 应用程序：

```javascript
var express = require('express')
var routes = require('./routes')
var user = require('./routes/user')
var http = require('http')
var path = require('path')

var app = express()

// 所有环境
app.set('port', process.env.PORT || 3000)
app.set('views', path.join(__dirname, 'views'))
app.set('view engine', 'pug')
app.use(express.favicon())
app.use(express.logger('dev'))
app.use(express.methodOverride())
app.use(express.session({ secret: 'your secret here' }))
app.use(express.bodyParser())
app.use(app.router)
app.use(express.static(path.join(__dirname, 'public')))

// 仅限开发环境
if (app.get('env') === 'development') {
  app.use(express.errorHandler())
}

app.get('/', routes.index)
app.get('/users', user.list)

http.createServer(app).listen(app.get('port'), function () {
  console.log('Express server listening on port ' + app.get('port'))
})
```

#### `package.json`

相应的版本 3 `package.json` 文件可能如下所示：

```json
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "3.12.0",
    "pug": "*"
  }
}
```

### 过程

开始迁移过程，通过以下命令安装 Express 4 应用程序所需的中间件，并将 Express 和 Pug 更新为各自的最新版本：

```console
$ npm install serve-favicon morgan method-override express-session body-parser multer errorhandler express@latest pug@latest --save
```

对 `app.js` 进行以下更改：

1. 内置的 Express 中间件函数 `express.favicon`、`express.logger`、`express.methodOverride`、`express.session`、`express.bodyParser` 和 `express.errorHandler` 不再在 `express` 对象上可用。您必须手动安装它们的替代品并在应用程序中加载它们。
2. 不再需要加载 `app.router` 函数。它不是有效的 Express 4 应用程序对象，因此删除 `app.use(app.router);` 代码。
3. 确保中间件函数以正确的顺序加载 - 在加载应用程序路由后加载 `errorHandler`。

### 版本 4 应用程序

#### `package.json`

运行上述 `npm` 命令将更新 `package.json` 如下：

```json
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "body-parser": "^1.5.2",
    "errorhandler": "^1.1.1",
    "express": "^4.8.0",
    "express

-session": "^1.7.2",
    "pug": "^2.0.0",
    "method-override": "^2.1.2",
    "morgan": "^1.2.2",
    "multer": "^0.1.3",
    "serve-favicon": "^2.0.1"
  }
}
```

#### `app.js`

然后，删除无效的代码，加载所需的中间件，并根据需要进行其他更改。`app.js` 文件将如下所示：

```javascript
var http = require('http')
var express = require('express')
var routes = require('./routes')
var user = require('./routes/user')
var path = require('path')

var favicon = require('serve-favicon')
var logger = require('morgan')
var methodOverride = require('method-override')
var session = require('express-session')
var bodyParser = require('body-parser')
var multer = require('multer')
var errorHandler = require('errorhandler')

var app = express()

// 所有环境
app.set('port', process.env.PORT || 3000)
app.set('views', path.join(__dirname, 'views'))
app.set('view engine', 'pug')
app.use(favicon(path.join(__dirname, '/public/favicon.ico')))
app.use(logger('dev'))
app.use(methodOverride())
app.use(session({
  resave: true,
  saveUninitialized: true,
  secret: 'uwotm8'
}))
app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended: true }))
app.use(multer())
app.use(express.static(path.join(__dirname, 'public')))

app.get('/', routes.index)
app.get('/users', user.list)

// 在加载路由后加载错误处理中间件
if (app.get('env') === 'development') {
  app.use(errorHandler())
}

var server = http.createServer(app)
server.listen(app.get('port'), function () {
  console.log('Express server listening on port ' + app.get('port'))
})
```

除非需要直接使用 `http` 模块（socket.io/SPDY/HTTPS），不再需要加载它，应用程序可以简单地以如下方式启动：

```javascript
app.listen(app.get('port'), function () {
  console.log('Express server listening on port ' + app.get('port'))
})
```

### 运行应用程序

迁移过程已经完成，现在应用程序是一个 Express 4 应用程序。要确认，通过以下命令启动应用程序：

```console
$ node .
```

加载 [http://localhost:3000](http://localhost:3000/)，查看由 Express 4 渲染的主页。

## 升级到 Express 4 应用生成器

用于生成 Express 应用程序的命令行工具仍然是 `express`，但要升级到新版本，必须先卸载 Express 3 应用生成器，然后安装新的 `express-generator`。

### 安装

如果您已经在系统上安装了 Express 3 应用生成器，必须卸载它：

```console
$ npm uninstall -g express
```

根据文件和目录权限配置的情况，您可能需要使用 `sudo` 运行此命令。

现在安装新的生成器：

```console
$ npm install -g express-generator
```

根据文件和目录权限配置的情况，您可能需要使用 `sudo` 运行此命令。

现在，您系统上的 `express` 命令已经更新为 Express 4 生成器。

### 应用生成器的变更

命令选项和用法在很大程度上保持不变，但有以下例外：

- 移除了 `--sessions` 选项。
- 移除了 `--jshtml` 选项。
- 添加了 `--hogan` 选项以支持 [Hogan.js](http://twitter.github.io/hogan.js/)。

### 示例

执行以下命令以创建一个 Express 4 应用程序：

```console
$ express app4
```

如果查看 `app4/app.js` 文件的内容，您会注意到除了 `express.static` 之外，应用程序所需的所有中间件函数都被加载为独立模块，而 `router` 中间件不再在应用程序中显式加载。

您还会注意到，`app.js` 文件现在是一个 Node.js 模块，与旧

版本的 Express 3 应用程序不同，其中 `app.listen()` 被调用以启动服务器，而不是通过 `http.createServer()`。

## 结论

​		Express 4 是一个重要的升级，引入了许多变化和新特性。尽管升级过程可能会有一些复杂性，但通过逐步进行，并参考官方文档和示例，您可以顺利地将现有的 Express 3 应用程序迁移到 Express 4，从而利用新的功能和改进。在迁移过程中，确保备份应用程序代码和配置，以防万一需要还原。如果需要使用 Express 3 的功能，您可以继续使用旧版本，但为了充分利用最新的功能和安全性，推荐尽早迁移到 Express 4。