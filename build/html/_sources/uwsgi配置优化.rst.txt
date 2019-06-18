基础配置
=========

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

