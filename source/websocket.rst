Websocket
================================

Introdution
-----------------

决定手头的工作是否需要使用WebSocket技术的方法很简单：

- 你的应用提供多个用户相互交流吗？
- 你的应用是展示服务器端经常变动的数据吗？

Django Websocket support -- channels
----------------------------------------
- A channel is a mailbox where messages can be sent to. Each channel has a name. 
	Anyone who has the name of a channel can send a message to the channel.
- A group is a group of related channels. A group has a name. 
	Anyone who has the name of a group can add/remove a channel to the group by name and
	send a message to all channels in the group. It is not possible to enumerate
	what channels are in a particular group.

HTTP
---------

keep-alive
~~~~~~~~~~~~~~~~~~

我们知道HTTP协议采用“请求-应答”模式，当使用普通模式，即非KeepAlive模式时，每个请求/应答客户和服务器都要新建一个连接，完成之后立即断开连接（HTTP协议为无连接的协议）；
当使用Keep-Alive模式（又称持久连接、连接重用）时，Keep-Alive功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接。

http 1.0中默认是关闭的，需要在http头加入"Connection: Keep-Alive"，才能启用Keep-Alive；http 1.1中默认启用Keep-Alive，如果加入"Connection: close"，
才关闭。目前大部分浏览器都是用http1.1协议，也就是说默认都会发起Keep-Alive的连接请求了，所以是否能完成一个完整的Keep-Alive连接就看服务器设置情况。

Keep-Alive模式，客户端如何判断请求所得到的响应数据已经接收完成（或者说如何知道服务器已经发生完了数据）？我们已经知道了，
Keep-Alive模式发送玩数据HTTP服务器不会自动断开连接，所有不能再使用返回EOF（-1）来判断（当然你一定要这样使用也没有办法，可以想象那效率是何等的低）！
下面我介绍两种来判断方法。

- 使用消息首部字段Conent-Length
- 使用消息首部字段Transfer-Encoding


HTTP的生命周期通过 Request 来界定，也就是发送一次 Request，收到一次 Response ，那么在 HTTP1.0 中，这次HTTP请求就结束了

在HTTP1.1中进行了改进，使得有一个keep-alive，也就是说，在一个HTTP连接中，可以发送多个Request，接收多个Response。但是请记住 Request = Response ， 在HTTP中永远是这样，也就是说一个request只能有一个response。而且这个response也是被动的，不能主动发起。

而对于websocket来说，在HTTP的握手基础上建立起链接，服务器端可以主动的向客户端发送数据。


nginx 配置
--------------

::

	    location /ws {
        proxy_pass http://192.168.198.207:16789;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host $server_name;
    }

否则当做http请求处理

.. Note:: nginx转到`websocket`的协议虽然是*http*的，但是内部`daphne`应该是有所转化的

apache 配置
---------------------

:: 
	
	LoadModule proxy_module modules/mod_proxy.so  
    LoadModule proxy_http_module modules/mod_proxy_http.so
    LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so

    ProxyPass /ws ws://ip:port/ws

    ProxyPass / http://127.0.0.1:8080/