Redis
=================

Base Command
---------------------

- `SELECT index` 选择数据库，一般的index in [0, 16), 对应`127.0.0.1:6379/index`
- `KEYS pattern` ==> `KEYS *` 获取所有键值
- `TTL` 获取键到期的剩余时间(秒)



django-redis
---------------------

django-redis==4.9.0 与reids-py >= 3.0.0 不兼容, ``https://github.com/niwinz/django-redis/issues/342``,
在django-redis==4.10.0 中修复
