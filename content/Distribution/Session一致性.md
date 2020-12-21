# session

服务器为每个用户创建一个会话，存储用户的相关信息，以便多次请求能够定位到同一个上下文。Web开发中，web-server可以自动为同一个浏览器的访问用户自动创建session，提供数据存储功能。最常见的，会把用户的登录信息、用户信息存储在session中，以保持登录状态。

# session同步

**思路**：多个web-server之间相互同步session，这样每个web-server之间都包含全部的session

**优点**：web-server支持的功能，应用程序不需要修改代码

**不足**：session的同步需要数据传输，占**内网带宽**，有时延。所有web-server都包含所有session数据，数据量受内存限制，无法水平扩展。有更多web-server时要歇菜

# 反向代理hash一致性

**四层代理hash**

反向代理层使用用户ip来做hash，以保证同一个ip的请求落在同一个web-server上

**七层代理hash**

反向代理使用http协议中的某些业务属性来做hash，例如sid，city_id，user_id等，能够更加灵活的实施hash策略，以保证同一个浏览器用户的请求落在同一个web-server上

**优点**：只需要改nginx配置，不需要修改应用代码。负载均衡，只要hash属性是均匀的，多台web-server的负载是均衡的。可以支持web-server水平扩展（session同步法是不行的，受内存限制）

**不足：**如果web-server重启，一部分session会丢失，产生业务影响，例如部分用户重新登录。如果web-server水平扩展，rehash后session重新分布，也会有一部分用户路由不到正确的session

# 后端统一存储

将session存储在web-server后端的存储层，数据库或者缓存。

没有安全隐患。可以水平扩展，数据库/缓存水平切分即可。web-server重启或者扩容都不会有session丢失

 增加了一次网络调用，并且需要修改应用代码

# 完全不用 Session

使用 JWT Token 储存用户身份，然后再从数据库或者 cache 中获取其他的信息。这样无论请求分配到哪个服务器都无所谓。

