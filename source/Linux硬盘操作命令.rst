Linux 硬盘命令
========================

df: 用于显示磁盘分区上的可使用的磁盘空间.常用``df -h[a]`` ::
    Filesystem                   Size  Used Avail Use% Mounted on
    udev                         7.8G     0  7.8G   0% /dev
    tmpfs                        1.6G   59M  1.5G   4% /run
    /dev/mapper/ubuntu--vg-root  228G   18G  199G   8% /
    tmpfs                        7.8G  188K  7.8G   1% /dev/shm
    tmpfs                        5.0M  4.0K  5.0M   1% /run/lock
    tmpfs                        7.8G     0  7.8G   0% /sys/fs/cgroup
    /dev/nvme0n1p1               720M  274M  410M  41% /boot
    /dev/sda1                    1.8T  6.5G  1.7T   1% /data
    tmpfs                        1.6G   32K  1.6G   1% /run/user/108
    tmpfs                        1.6G     0  1.6G   0% /run/user/1000

du: 对文件和目录磁盘使用的空间的查看.未指定参数会递归显示当前文件夹下所有文件的大小.常用
``du -h[s] [file]`` 
``du -h --max-depth=1``
``du --max-depth=1 | sort -nk 1``
::
    ...
    6.3M    ./.git
    44K ./build/html/_sources
    124K    ./build/html/_static/css
    5.4M    ./build/html/_static/fonts/Lato
    812K    ./build/html/_static/fonts/RobotoSlab
    8.9M    ./build/html/_static/fonts
    28K ./build/html/_static/js
    9.6M    ./build/html/_static
    9.7M    ./build/html
    132K    ./build/doctrees
    9.9M    ./build
    36K ./source/_build/html/_sources
    124K    ./source/_build/html/_static/css
    5.4M    ./source/_build/html/_static/fonts/Lato
    812K    ./source/_build/html/_static/fonts/RobotoSlab
    8.9M    ./source/_build/html/_static/fonts
    28K ./source/_build/html/_static/js
    9.5M    ./source/_build/html/_static
    112K    ./source/_build/html/.doctrees
    9.8M    ./source/_build/html
    9.8M    ./source/_build
    9.9M    ./source
    26M .


.. Tip:: 有时候 ``tar`` 的时候打的包太大，有的软件传输文件又有大小限制，所有先用 ``du -h`` 看一下哪些文件比较大，又是用不到的，
    使用 ``tar --exclude path/.git --exclude *.pyc ...`` 减小压缩包的大小


fdisk: 用于观察硬盘实体使用情况，也可对硬盘分区.常用``fdisk -l`` ::
    Disk /dev/nvme0n1: 232.9 GiB, 250059350016 bytes, 488397168 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xffd70372

    Device         Boot   Start       End   Sectors   Size Id Type
    /dev/nvme0n1p1 *       2048   1499135   1497088   731M 83 Linux
    /dev/nvme0n1p2      1501182 488396799 486895618 232.2G  5 Extended
    /dev/nvme0n1p5      1501184 488396799 486895616 232.2G 8e Linux LVM


    Disk /dev/sda: 1.8 TiB, 2000398934016 bytes, 3907029168 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 4096 bytes
    I/O size (minimum/optimal): 4096 bytes / 4096 bytes
    Disklabel type: dos
    Disk identifier: 0x598f1b92

    Device     Boot Start        End    Sectors  Size Id Type
    /dev/sda1        2048 3907029167 3907027120  1.8T 83 Linux


    Disk /dev/mapper/ubuntu--vg-root: 231.2 GiB, 248260853760 bytes, 484884480 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/ubuntu--vg-swap_1: 980 MiB, 1027604480 bytes, 2007040 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


mkfs: 用于在设备上（通常为硬盘）创建Linux文件系统。mkfs本身并不执行建立文件系统的工作，而是去调用相关的程序来执行。
::
    mkfs -t ext4 /dev/sda


mount: 用于加载文件系统到指定的加载点。此命令的最常用于挂载cdrom，使我们可以访问cdrom中的数据，因为你将光盘插入cdrom中，
    Linux并不会自动挂载，必须使用Linux mount命令来手动完成挂载. 不会开机自动挂载磁盘,即是一次性的.
::
    mount /dev/sda /data


``/etc/fstab``是用来存放文件系统的静态信息的文件.当系统启动的时候，系统会自动地从这个文件读取信息，
并且会自动将此文件中指定的文件系统挂载到指定的目录(使得能够开机自动挂载磁盘).

格式::
    <file system>   <dir>   <type>  <options>   <dump>  <pass>

    <dump> dump 工具通过它决定何时作备份. dump 会检查其内容，并用数字来决定是否对这个文件系统进行备份。 
    允许的数字是 0 和 1 . 0 表示忽略， 1 则进行备份。大部分的用户是没有安装 dump 的 ，对他们而言 <dump> 应设为 0。

    <pass> fsck 读取 <pass> 的数值来决定需要检查的文件系统的检查顺序. 允许的数字是 0, 1, 和 2. 
    根目录应当获得最高的优先权 1, 其它所有需要被检查的设备设置为 2. 0 表示设备不会被 fsck 所检查.
    如:
    /dev/sda   /data   ext4    defaults    0   0


lsblk: 用于列出所有可用块设备的信息，而且还能显示他们之间的依赖关系，但是它不会列出RAM盘的信息. 常用``lsblk -f``
::
    NAME                  FSTYPE      LABEL UUID                                   MOUNTPOINT
    sda                                                                            
    └─sda1                ext4              3fa046f2-31e2-460d-b9ac-d8d53b01da11   /data
    nvme0n1                                                                        
    ├─nvme0n1p5           LVM2_member       lHrhEi-QARE-Zd0Z-Kk1m-wN4D-pIoG-oz4jyp 
    │ ├─ubuntu--vg-swap_1 swap              15ff3d96-31b6-4f7d-aa32-31e123fffef8   [SWAP]
    │ └─ubuntu--vg-root   ext4              99bde79a-d237-4fc1-a82c-2a69f1af8d25   /
    ├─nvme0n1p1           ext2              a186cdb4-561f-4bd1-8044-c707ce8a0e57   /boot
    └─nvme0n1p2 

