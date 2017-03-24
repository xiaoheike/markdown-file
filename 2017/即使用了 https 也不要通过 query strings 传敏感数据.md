A common question we hear is “Can parameters be safely passed in URLs to secure web sites? ” The question often arises after a customer has looked at an HTTPS request in HttpWatch and wondered who else can see this data.
我们经常听到的一个常见问题是：“`URL` 中的参数是否可以安全地传递到安全网站？”这个问题常常出现在客户看了 `HttpWatch` 捕获的 `HTTPS` 请求后，想知道还有谁可以看到这些数据。

For example, let’s pretend to pass a password in a query string parameter using the following secure URL:
***https://www.httpwatch.com/?password=mypassword***
例如，假设在一个查询中，使用如下安全的 `URL` 传递密码字符串：
***https://www.httpwatch.com/?password=mypassword***

HttpWatch is able to show the contents of a secure request because it is integrated with the browser and can view the data before it is encrypted by the SSL connection used for HTTPS requests:
`HttpWatch` 能够显示安全请求的内容，因为它与浏览器集成，因此它能够在 `HTTPS` 请求的 `SSL` 连接对数据加密之前查看数据。
【图片1】 


If you look in a network sniffer, like Network Monitor, at the same request you would just see the encrypted data going backwards and forwards. No URLs, headers or content is visible in the packet trace::
如果你使用网络嗅探器查看，例如 `Newwork Monitor`，对于同一个请求，你只能够查阅加密之后的数据。在数据包跟踪中没有可见的网址，标题或内容：
【图片2】
 
You can rely on an HTTPS request being secure so long as:
- No SSL certificate warnings were ignored
- The private key used by the web server to initiate the SSL connection is not available outside of the web server itself.
您可以信任 `HTTPS` 请求是安全的，只要：
- 未忽略任何SSL证书警告
- Web 服务器用于启动 SSL 连接的私钥在 Web 服务器本身之外不可用。

So at the network level, URL parameters are secure, but there are some other ways in which URL based data can leak:
因此，在网络层面，`URL` 参数是安全的，但是还有一些其他基于 `URL` 泄漏数据的方法：

1. URLs are stored in web server logs – typically the whole URL of each request is stored in a server log. This means that any sensitive data in the URL (e.g. a password) is being saved in clear text on the server. Here’s the entry that was stored in the httpwatch.com server log when a query string was used to send a password over HTTPS:
**2009-02-20 10:18:27 W3SVC4326 WWW 208.101.31.210 GET /Default.htm password=mypassword 443 ...
It’s generally agreed that storing clear text passwords is never a good idea even on the server.
1. `URL` 存储在 Web 服务器日志中--通常每个请求的完整 `URL` 都被存放在服务器日志中。这意味着 `URL` 中的任何敏感数据（例如密码）会以明文形式保存在服务器上。以下是使用查询字符串通过 `HTTPS` 发送密码时存储在 `httpwatch.com` 服务器日志中的条目：
**2009-02-20 10:18:27 W3SVC4326 WWW 208.101.31.210 GET /Default.htm password=mypassword 443 ...
通常认为即使是在服务器上，[存储明文密码从来都不是好想法](http://www.codinghorror.com/blog/archives/000953.html)

2.	URLs are stored in the browser history – browsers save URL parameters in their history even if the secure pages themselves are not cached. Here’s the IE history displaying the URL
parameter:
2. `URL` 存储在浏览器历史记录中--即使安全网页本身未缓存，浏览器也会将 `URL` 参数保存在其历史记录中。以下是 `IE` 的历史记录，显示了 `URL` 的请求参数：
【图片3】

Query string parameters will also be stored if the user creates a bookmark.
如果用户创建书签，查询字符串参数也将被存储。

3.	URLs are passed in Referrer headers – if a secure page uses resources, such as javascript, images or analytics services, the URL is passed in the Referrer request header of each embedded request. Sometimes the query string parameters may be delivered to and stored by third party sites. In HttpWatch you can see that our password query string parameter is being sent across to Google Analytics:
3. `URL` 在 `Referrer` 请求头中被传递--如果一个安全网页使用资源，例如 `javascript`，图片或者分析服务，`URL` 将通过 `Referrer` 请求头传递到每一个嵌入对象。有时，查询字符串参数可能被传递并存放在第三方站点。在 `HttpWatch` 中，你可以看到我们的密码字符串正被发送到 `Google Analytics`：
【图片4】

 
**Conclusion**
**结论**

The solution to this problem requires two steps:
- Only pass around sensitive data if absolutely necessary. Once a user is authenticated it is best to identify them with a session ID that has a limited lifetime.
- Use non-persistent, session level cookies to hold session IDs and other private data.
解决这个问题需要两步：
- 只有在绝对必要的情况下传递敏感数据。一旦用户被认证，最好使用具有有限生命周期的会话 `ID` 来标识它们。

The advantage of using session level cookies to carry this information is that:
- They are not stored in the browsers history or on the disk
- They are usually not stored in server logs
- They are not passed to embedded resources such as images or javascript libraries
- They only apply to the domain and path for which they were issued
使用会话层级的 `cookies` 传递信息的优点是：
- 它们不会存储在浏览器历史记录中或磁盘上
- 它们通常不存储在服务器日志中
- 它们不会传递到嵌入式资源，例如图片或 `JavaScript` 库
- 它们仅适用于请求它们的域和路径

Here’s an example of the ASP.NET session cookie that is used in our online store to identity a user:
以下是我们的在线商店中，用于识别用户的 `ASP.NET` 会话 `cookie` 示例：
【图片5】

Notice that the cookie is limited to the domain store.httpwatch.com and it expires at the end of the browser session (i.e. it is not stored to disk).
请注意，`cookie` 被限制在域 `store.httpwatch.com`，并且在浏览器会话结束时过期（即不会存储到磁盘）。

You can of course use query string parameters with HTTPS, but don’t use them for anything that could present a security problem. For example, you could safely use them to identity part numbers or types of display like ‘accountview’ or ‘printpage’, but don’t use them for passwords, credit card numbers or other pieces of information that should not be publicly available.
你当然可以通过 `HTTPS` 传递查询字符串，但是不要在可能出现安全问题的场景下使用。例如，你可以安全的使用它们显示部分数字或者类型，像 `accountview` 或者 `printpage`，但是不要使用它们传递密码，信用卡号码或者其他不应该公开的信息。

