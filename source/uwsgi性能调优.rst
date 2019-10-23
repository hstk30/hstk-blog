uwsgi 性能调优
=================

影响网络性能的系统参数：
---------------------

max file descriptior number
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

系统级限制: ``cat /proc/sys/fs/file-max``
修改：``/etc/sysctl.conf``中增加``fs.file-max = xxxxxxxxxx``
生效：``sysctl -p``

用户级限制: ``ulimit -n``是设置**当前shell**的**当前用户**所有进程能打开的最大文件数量.``ulimit -Sn`` ``ulimit -Hn``
修改：``/etc/security/limits.conf``中增加::
	# 具体信息直接参考文件中的注释
	# <domain>	<type>	<item>	<value>
	# <type> nofile - max number of open files
	* soft nofile 2000	# 所有用户最大可打开文件数量的软限制为2000
	* hard nofile 2048


.. Note:: 在**docker**里起的镜像可能与当前用户可打开的最大文件数量不同,需要具体查看.



listen queue size: net.core.somaxconn
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

net.core.somaxconn是Linux中的一个kernel参数，表示socket监听（listen）的backlog上限。
什么是backlog呢？backlog就是socket的监听队列，当一个请求（request）尚未被处理或建立时，他会进入backlog。
而socket server可以一次性处理backlog中的所有请求，处理后的请求不再位于监听队列中。
当server处理请求较慢，以至于监听队列被填满后，新来的请求会被丢弃。

``cat /proc/sys/net/core/somaxconn`` Linux 默认设置为128

修改：``/etc/sysctl.conf``中增加``net.core.somaxconn = 2048``
生效：``sysctl -p``

error::
	
	192.168.198.64 - - [17/Jun/2019:17:10:59 +0800] "GET /xx/xxx HTTP/1.1" 499 0 "-" "python-requests

The problem is that clients abort the connection and then Nginx closes the connection without telling uwsgi to abort. 
Then when uwsgi comes back with the result the socket is already closed. 
Nginx writes a 499 error in the log and uwsgi throws a IOError.

Basically I know that nginx is aware of the premature abort/close because it defaults to uwsgi_ignore_client_abort to "off", and you get nginx 499 errors in your nginx logs when requests are aborted/closed before the response is sent. Once uwsgi finishes processing the request it throws an "IO Error" when it goes to return the response to nginx.

Turning uwsgi_ignore_client_abort to "on" just makes nginx unaware of the abort/close, and removes the uwsgi "IO Errors" because uwsgi can still write back to nginx.


关于uwgsi优化配置注意事项：
----------------------------

1. 线程不能随意的开启，如果非要开启的话，记得需要开启异步处理项,如下::

    enable-threads = true
    threads = 4
    async = 4

2. <listen>2048</listen>项是提高并发的关键参数，参数过小的，并发大的时候请求就容易被丢弃处理修改
    这个值的方式是：可以通过 /proc/sys/net/somaxconn 和 /proc/sys/net/ipv4/tcp_max_syn_backlog 对TCP
    socket进行调整修改系统参数修改/etc/sysctl.conf文件,添加或者修改这几个参数值对于一个经常处理新连接的高负载web服务环境来说，
    默认的 128 太小了net.core.somaxconn = 262144表示SYN队列的长度，默认为1024，加大队列长度为8192，
    可以容纳更多等待连接的网络连接数net.ipv4.tcp_max_syn_backlog =8192网卡设备将请求放入
    队列的长度net.core.netdev_max_backlog = 65536 修改完成之后要记得 sysctl -p 重新加载参数

3. 增加（或减少）超时时间是重要的，修改socket监听队列大小也是如此。
4. 考虑线程。如果你不需要线程，那么不要启用它们。
5. 总是记住在生产环境上启用master进程。见 ProcessManagement 。
6. 增加worker不是意味着“增加性能”，因此，基于你的应用的性质，为 workers 选项选择一个合适的值 
    (受IO限制，受CPU限制，IO等待……),processes = 2 * cpucores 最佳processes 设置值

7. post-buffering的配置可以有效帮应用自动读取相关POST提交请求体信息.打开http body缓冲, 如果HTTP
    body的大小超过指定的限制,那么就保存到磁盘上.

8. 检查应用的内存使用 memory-report
9. 建议不要以root用户运行uWSGI实例。如果非要你可以作为建议确保确保使用 uid 和 gid 选项来移除权限。
10. 如果 (Linux) 服务器似乎有大量的idle状态的worker，但性能仍然不佳，需要看看 ip_conntrack_max 系统变量 
    (/proc/sys/net/ipv4/ip_conntrack_max) 的值，然后增加它以看看是否有帮助。
11. 如果uwsgi日志中开始收到”invalid request block size”，这可能意味着你需要更大的缓存了。
    通过buffer-size 选项来增加它（最大值为65535）。
12. async: 打开异步模式

13. buffer-size 为分析uwsgi包准备的内部缓冲区大小,默认是4K

    - buffer-size 32768
    - s 指定unix socket路径或者tcp 的地址和端口号
    - p worker进程数量
    - d daemonize

14. 在10s钟内没有活动就关闭连接:–socket-timeout 10 (默认是4s)

参考
------

- `Linux之TCPIP内核参数优化`_
- `increate set open file limits in linux`_


.. _Linux之TCPIP内核参数优化:
   https://www.cnblogs.com/fczjuever/archive/2013/04/17/3026694.html
.. _increate set open file limits in linux:
   https://www.tecmint.com/increase-set-open-file-limits-in-linux
.. _linux最大文件句柄数量总结:
   https://jameswxx.iteye.com/blog/2096461
.. _Linux可打开最大文件数:
   https://ivanzz1001.github.io/records/post/linuxops/2018/03/20/linux-openfile-max#2-%E4%BF%AE%E6%94%B9%E6%96%B9%E6%B3%95