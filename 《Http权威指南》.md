# Http权威指南

## URI与URL
URI（统一资源标识符）分为：
- URL（统一资源定位符）
- URN（统一资源名），试验阶段，未大范围使用

## URL语法
基本格式：
```
<scheme>://<user>:<password>@<host>:<port>/<path>;<key=value>?<query>#<frag>
```
某些协议如ftp需要<key=value>参数；<query>为请求查询；<frag>指向文档的特定章节
</path>包含特殊字符需进行“%+2位16进制ASCII”转义："%20"对应ASCII32的“space空格”

## 媒体类型-MIME type
MIME type，最初设计为解决邮件传输问题，http采用它来描述多媒体类型：
- HTML网页：text/html
- ASCII文档：text/plain

## 结构
<img width="487" alt="截屏2023-03-06 23 04 47" src="https://user-images.githubusercontent.com/56404426/223460116-f00fccbe-b0a8-4381-a55d-856ab5f68b2b.png">

首部以一个空行结束

## 30X重定向状态码
- 301 Move Permanently， Location首部应包含资源所处的URL
- 304 Not Modify, 资源近期未修改，响应不应该包括实体
- 302 Found、303 See Other与307

Temporary Redirect：Http1.0规范客户端发送POST请求后收到302响应，应询问用户进行下一步动作，但实际上厂商在未询问用户情况下自动发起GET请求Location首部的URL，因此Http1.1加入了303为自动发送GET请求，同时为了兼容Http1.0客户端，服务端在发起303时也会替换为302，307则是继承了原先的302

## Connection首部与keep-alive
Connection可以承载3种不同类型的标签
- 此连接存在的首部
- 任意标签值
- close， 此操作完成后需关闭这条连接

代理转发包含Connection首部请求时需要**删除**连接中Connection存在的首部**及**Connection本身

http1.0使用“Connection:keep-alive”来**请求**持久会话，服务端需要在响应中添加“Connection:keep-alive”才能保持会话，可选的响应首部“Keep-Alive:max=<value1>, timeout=<value2>”代表了服务器预估为最多max个事务或最多timeout秒保持连接

http1.1默认所有连接都是持久连接，除非显示地添加“Connection:close”，但服务端和客户端可以随时关闭空闲连接

## 代理和网关
严格来说，代理连接多个使用相同协议的应用程序；网关连接的则是多个使用不同协议的应用程序。实际上很多商用代理具备协议转换的功能。
代理根据部署方式分为：
- 出口代理：控制和过滤流量
- （访问）入口代理：聚合用户请求，缓存常用文档
- 反向代理：服务器负载均衡
- 网络交换代理：通过缓存减轻因特网节点的拥塞

## Via首部
报文每经过一个代理或网关时，添加该节点信息到Via首部末尾

## 公共缓存（代理缓存）
请求到达缓存，无缓存或缓存过期则询问服务器或父代理并缓存文档和**响应首部**

缓存过期进行的再验证：
- If-Modified-Since: \<date> 若指定日期后文档未发生修改，返回304请求
- If-None-Match: \<Tag> 配合“ETag”首部，文档版本更新则回送文档
服务器控制缓存使用Cache-Control首部，优先级递减
- no-store：禁用缓存对响应的复制
- no-cache：缓存可以复制文档，但与服务器进行新鲜度再验证之后才能提供文档
- must-revalidate：对于过期的文档，缓存应该与服务器进行再验证
- max-age：自文档生成后新鲜持续时间，单位秒

客户端对缓存的控制Cache-Control:
- max-stale: 缓存可以提供过期的文档，若指定了时间，则表示过期时间不能超过该秒数
- min-fresh=\<s>：指定秒数内保持最新的文档
- max-age：缓存无法返回时间大于给定时间的文档
- no-cache：客户端不接受缓存未再验证的文档
- no-store：告知缓存删除文档