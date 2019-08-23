Linux 文件目录
=======================

文件系统层次结构标准(Filesystem Hierarchy Standard, FHS)(`维基百科`_)

__
.. _维基百科: https://zh.wikipedia.org/wiki/文件系统层次结构标准


目录结构
---------------

- /bin/ 需要在单用户模式可用的必要命令（可执行文件）；面向所有用户，例如： ``cat``、 ``ls``、 ``cp``。
- /boot/ 引导程序文件，例如： ``kernel``、``initrd``；时常是一个单独的分区
- /dev/ 必要设备, 例如：``/dev/null``, ``/dev/tty``
- /etc/ 特定主机，系统范围内的配置文件。关于这个名称当前有争议。在贝尔实验室关于UNIX实现文档的早期版本中，/etc 被称为etcetera，这是由于过去此目录中存放所有不属于别处的所有东西（然而，FHS限制/etc存放静态配置文件，不能包含二进制文件）。自从早期文档出版以来，目录名称已被以各种方式重新称呼。最近的解释包括反向缩略语如："可编辑的文本配置"（英文 "Editable Text Configuration"）或"扩展工具箱"（英文 "Extended Tool Chest")。
