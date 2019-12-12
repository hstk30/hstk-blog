TCP 
-----------------


Connection reset by peer
=================================

::
	This means that a TCP RST was received and the connection is now closed. This occurs when a packet is sent from your end of the connection but the other end does not recognize the connection; it will send back a packet with the RST bit set in order to forcibly close the connection.

	This can happen if the other side crashes and then comes back up or if it calls close() on the socket while there is data from you in transit, and is an indication to you that some of the data that you previously sent may not have been received.

	It is up to you whether that is an error; if the information you were sending was only for the benefit of the remote client then it may not matter that any final data may have been lost. However you should close the socket and free up any other resources associated with the connection.

	这意味着已收到TCP RST，并且现在已关闭连接。 当从您的连接端发送了一个数据包，但另一端无法识别该连接时，就会发生这种情况。 它将发回一个RST位置1的数据包，以强制关闭连接。
	如果另一端崩溃然后又重新启动，或者在传输过程中有数据从您的套接字上调用close（）时，可能会发生这种情况，这表明您以前发送的某些数据可能没有 已收到。
	这是否是错误取决于您自己； 如果您发送的信息仅是为了远程客户端的利益，那么可能丢失任何最终数据都可能无关紧要。 但是，您应该关闭套接字并释放与该连接关联的所有其他资源。


Broken Pipe
=====================

向一个已经关闭的通道写数据时会出现这个 Error 


socket program: send http request
================================================

::
	import socket
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((host, ip))

	body = b"""GET /workers/patients/overview HTTP/1.1\r\nHost: 127.0.0.1:8000\r\nX-USER-TOKEN: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxNiwidXNlcl9uYW1lIjoiXHU2ZDRiXHU4YmQ1XHU3YmExXHU3NDA2XHU1NDU4IiwidXNlcl9yb2xlIjoiQWRtaW4iLCJpYXQiOjE1NzQzMDAyMTksImV4cCI6MTU3NDM0MzQxOX0.ikmLkU6usmhPoNA_cEtNTEsmUDaJee__yiG6GKc9UN\r\nUser-Agent: PostmanRuntime/7.19.0\r\nAccept: */*\r\nCache-Control: no-cache\r\nPostman-Token: ef98ebbc-6a41-42d0-adae-598c9093bf6a,70996d6b-5ce0-4f32-b31d-9fb483c1fb65\r\nHost: 127.0.0.1:8000\r\nAccept-Encoding: gzip, deflate\r\nConnection: keep-alive\r\ncache-control: no-cache\r\n\r\n"""

	s.sendall(body)
	data = s.recv(1024)
	print(data)


之所以手写原生的http 请求，是想捕捉一些被框架隐藏的异常。如上面提到的`Connection reset by peer` 和 `broken pipe`。
一般http 的客户端都会有个连接池`ConnectionPool`来复用TCP 连接，而不是每个请求都重新创建一个TCP 连接。而如果在连接池中的
socket 对象的远端连接对象突然宕机了，在连接池没有删除这个socket 对象之前，这个远端对象有恢复了，那么再请求这个远端对象，这个
socket 对象是否会重置？？


telnet send http request
=====================================

::
	> telnet ip port
	接下来利用左侧ctrl+]来进入回显模式，再按一次回车进入输入模式，输入HTTP REQUEST BODY,回车


