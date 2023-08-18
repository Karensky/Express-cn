# 在反向代理后面使用 Express

当在反向代理后运行 Express 应用程序时，一些 Express API 可能会返回与预期不同的值。为了进行调整，可以使用 `trust proxy` 应用程序设置来在 Express API 中公开反向代理提供的信息。最常见的问题是，暴露客户端 IP 地址的 Express API 可能会显示反向代理的内部 IP 地址而不是预期的客户端 IP 地址。

在配置 `trust proxy` 设置时，重要的是要了解反向代理的确切设置。由于此设置将信任请求中提供的值，因此必须确保 Express 中的设置与反向代理的操作方式相匹配。

应用程序设置 `trust proxy` 可以设置为以下表中列出的值之一。

| 类型    | 值                                                           |
| ------- | ------------------------------------------------------------ |
| 布尔值  | 如果为 `true`，则客户端的 IP 地址被理解为 `X-Forwarded-For` 标头中最左边的条目。如果为 `false`，则该应用程序被理解为直接面向客户端，客户端的 IP 地址将从 `req.socket.remoteAddress` 获取。这是默认设置。当设置为 `true` 时，重要的是确保最后一个可信任的反向代理已删除/覆盖以下所有 HTTP 标头：`X-Forwarded-For`、`X-Forwarded-Host` 和 `X-Forwarded-Proto`，否则客户端可能会提供任何值。 |
| IP 地址 | IP 地址、子网或 IP 地址和子网的数组，可信任其为反向代理。以下列表显示了预配置的子网名称：loopback - `127.0.0.1/8`、`::1/128`linklocal - `169.254.0.0/16`、`fe80::/10`uniquelocal - `10.0.0.0/8`、`172.16.0.0/12`、`192.168.0.0/16`、`fc00::/7`您可以通过以下任一方式设置 IP 地址：`app.set('trust proxy', 'loopback') // 指定单个子网 app.set('trust proxy', 'loopback, 123.123.123.123') // 指定子网和地址 app.set('trust proxy', 'loopback, linklocal, uniquelocal') // 以 CSV 形式指定多个子网 app.set('trust proxy', ['loopback', 'linklocal', 'uniquelocal']) // 以数组形式指定多个子网 `当指定 IP 地址或子网时，将从地址确定过程中排除这些 IP 地址或子网，最靠近应用程序服务器的不受信任的 IP 地址将被确定为客户端的 IP 地址。这通过检查 `req.socket.remoteAddress` 是否受信任来实现。如果是，那么会从右到左逐个检查 `X-Forwarded-For` 中的每个地址，直到找到第一个不受信任的地址。 |
| 数字    | 使用距 Express 应用程序最多 `n` 个跳数的地址。`req.socket.remoteAddress` 是第一个跳数，其余的从右到左在 `X-Forwarded-For` 标头中查找。值为 `0` 意味着第一个不受信任的地址将是 `req.socket.remoteAddress`，即没有反向代理。在使用此设置时，重要的是确保没有多个不同长度的路径到达 Express 应用程序，这样客户端可以少于配置的跳数，否则客户端可能会提供任何值。 |
| 函数    | 自定义信任实现。`app.set('trust proxy', function (ip) {  if (ip === '127.0.0.1' || ip === '123.123.123.123') return true // 受信任的 IP 地址  else return false }) ` |

启用 `trust proxy` 将产生以下影响：

- [req.hostname](https://www.expressjs.com.cn/en/api.html#req.hostname) 的值来自于 `X-Forwarded-Host` 标头中设置的值，可以由客户端或代理设置。
- `X-Forwarded-Proto` 可以由反向代理设置为告知应用程序是否为 `https`、`http` 或甚至是无效的名称。此值会反映在 [req.protocol](https://www.expressjs.com.cn/en/api.html#req.protocol) 中。
- [req.ip](https://www.expressjs.com.cn/en/api.html#req.ip) 和 [req.ips](https://www.expressjs.com.cn/en/api.html#req.ips) 的值是基于套接字地址和 `X-Forwarded-For` 标头填充的，从第一个不受信任的地址开始。

`trust proxy` 设置是使用 [proxy-addr](https://www.npmjs.com/package/proxy-addr) 包实现的。有关更多信息，请参阅其文档。