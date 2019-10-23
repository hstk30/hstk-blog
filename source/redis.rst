Redis
=================

Base Command
---------------------

- ``SELECT index`` 选择数据库，一般的index in [0, 16), 对应`127.0.0.1:6379/index`
- ``KEYS pattern`` ==> ``KEYS *`` 获取所有键值
- ``TTL`` 获取键到期的剩余时间(秒)
- ``MONITOR``  实时打印出 Redis 服务器接收到的命令，调试用. ``redis-cli -h xx -p xx monitor > redislog.log``记录一段时间的log.


config
------------------

protected-mode 是在没有显示定义 bind 地址（即监听全网断），又没有设置密码 requirepass
时，protected-mode 只允许本地回环 127.0.0.1 访问。
也就是说当开启了 protected-mode 时，如果你既没有显示的定义了 bind 监听的地址，同时又没有设置 auth 密码。那你只能通过 127.0.0.1 来访问 redis 服务。



msgpack
-------------------

``pip install msgpack``

::

	msgpack.unpackb(b'')


django-redis
---------------------

django-redis==4.9.0 与reids-py >= 3.0.0 不兼容, ``https://github.com/niwinz/django-redis/issues/342``,
在django-redis==4.10.0 中修复
