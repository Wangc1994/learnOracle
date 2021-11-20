# learnOracle

## oracle12C安装

### 准备工作

一台阿里云服务器（Linux）、oracle 12C压缩包（linuxx64_12201_database.zip）

### 安装过程

1. **先把oracle 12C压缩包上传到云服务器**
2. **操作系统软硬件检查**
    - 内存要求1G+，建议2GB以上
      - 使用`grep MemTotal /proc/meminfo`查询服务器内存
        ```
        grep MemTotal /proc/meminfo 
        MemTotal:        3677176 kB
        ```
    - 内核要求 <BR>
        - 查询系统位数命令： uname -m <BR>
        - 查询系统版本命令：cat /proc/version或 cat /etc/redhat-release或 lsb_release -id<BR>
        - 查询系统内核版本：uname -r<BR>

        ```
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# cat /proc/version 
        Linux version 4.18.0-193.28.1.el8_2.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 8.3.1 20191121 (Red Hat 8.3.1-5) (GCC)) #1 SMP Thu Oct 22 00:20:22 UTC 2020
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# uname -r
        4.18.0-193.28.1.el8_2.x86_64
        ```
    - 磁盘空间要求 <BR> 
      - 使用`df -h <文件夹名>`查看磁盘空间使用情况
        ```Xshell
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# df -h
        Filesystem      Size  Used Avail Use% Mounted on
        devtmpfs        1.8G     0  1.8G   0% /dev
        tmpfs           1.8G  4.0K  1.8G   1% /dev/shm
        tmpfs           1.8G  496K  1.8G   1% /run
        tmpfs           1.8G     0  1.8G   0% /sys/fs/cgroup
        /dev/vda1        80G   13G   68G  16% /
        tmpfs           360M     0  360M   0% /run/user/0
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# df -h /tmp
        Filesystem      Size  Used Avail Use% Mounted on
        /dev/vda1        80G   13G   68G  16% /
        ```
    注：Oracle安装目录中的/oradata目录用来存放数据文件，/tmp目录是根文件系统的一部分，而图中查询结果显示根目录还剩余68G，满足条件。
3. **安装前系统配置**
   - RPM要求
     - 使用`rpm -q package_name`检查RPM安装情况
        ```
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# rpm -q binutils compat-libcap1 compat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libaio libaio-devel libgcc libstdc++ libstdc++-devel libXext libXtst libX11 libXau libxcb libXi make sysstat
        binutils-2.30-73.el8.x86_64
        package compat-libcap1 is not installed
        package compat-libstdc++-33 is not installed
        gcc-8.3.1-5.1.el8.x86_64
        gcc-c++-8.3.1-5.1.el8.x86_64
        glibc-2.28-127.el8.x86_64
        glibc-devel-2.28-127.el8.x86_64
        package ksh is not installed
        libaio-0.3.112-1.el8.x86_64
        package libaio-devel is not installed
        libgcc-8.3.1-5.1.el8.x86_64
        libstdc++-8.3.1-5.1.el8.x86_64
        libstdc++-devel-8.3.1-5.1.el8.x86_64
        libXext-1.3.3-9.el8.x86_64
        package libXtst is not installed
        libX11-1.6.8-3.el8.x86_64
        libXau-1.0.9-3.el8.x86_64
        libxcb-1.13.1-1.el8.x86_64
        package libXi is not installed
        make-4.2.1-10.el8.x86_64
        sysstat-11.7.3-5.el8.x86_64
        ```
     - 使用命令`yum -y install package_name`安装缺失的RPM包
     - 创建wap分区
       - 使用`free -m`查看wap分区情况
        ```
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# free -m
                      total        used        free      shared  buff/cache   available
        Mem:           3590         722         188          30        2680        2565
        Swap:             0           0           0
        ```
       - 增加4G交换空间
        ```
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# dd if=/dev/zero of=/usr/swap bs=1024 count=4096000
        4096000+0 records in
        4096000+0 records out
        4194304000 bytes (4.2 GB, 3.9 GiB) copied, 27.1218 s, 155 MB/s
        ```
       - 设置交换空间

        ```
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# dd if=/dev/zero of=/usr/swap bs=1024 count=4096000
        4096000+0 records in
        4096000+0 records out
        4194304000 bytes (4.2 GB, 3.9 GiB) copied, 27.1218 s, 155 MB/s
        ```
       - 启动交换空间
        ```
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# mkswap /usr/swap
        mkswap: /usr/swap: insecure permissions 0644, 0600 suggested.
        Setting up swapspace version 1, size = 3.9 GiB (4194299904 bytes)
        no label, UUID=5917e508-384a-4e2f-8c21-e059d7d68cfb
        ```

       - 再次查看wap分区情况
        ```
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# swapon /usr/swap
        swapon: /usr/swap: insecure permissions 0644, 0600 suggested.
        [root@iZbp1itnf4brss8qb2rqhwZ ~]# free -m
                      total        used        free      shared  buff/cache   available
        Mem:           3590         703         108          30        2778        2604
        Swap:          3999           0        3999
        ```
    - 创建用户、用户组及安装目录<BR>
    *安装和运行Oracle数据库软件都需要使用指定用户组内的指定用户，用户为Oracle，出于安全考虑，用户组建为oinstall、dba，oinstall组中的成员用于管理Oracle数据库物理软件，dba组中的成员用于管理、操作数据库，具有sysdba权限。*
    ```
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# groupadd oinstall ----创建oracle用户组 
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# groupadd dba ----创建oracle用户组 
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# useradd -g oinstall -G dba oracle ----oracle加入新建的2个用户组 
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# passwd oracle ----设置oracle用户的密码  
    Changing password for user oracle.
    New password:Wang941220
    Retype new password:Wang941220
    passwd: all authentication tokens updated successfully.
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# mkdir -p /usr/oracle ----创建oracle安装目录
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# mkdir -p /opt/oracle/oracinstall ---创建racle安装文件所在目录
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# chown -R oracle:oinstall /usr/oracle ----更改oracle目录用户组  
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# chmod -R 775 /usr/oracle ----更改oracle目录权限 
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# chown -R oracle:oinstall /opt/oracle/oracinstall ----更改oracle安装文件所在目录的用户组
    [root@iZbp1itnf4brss8qb2rqhwZ ~]# chmod -R 755 /opt/oracle/oracinstall ----更改oracleracle安装文件所在目录的操作权限 
    ```
    - 配置系统内核参数
      - 编辑系统的内核参数：`vi /etc/sysctl.conf`，在文件的末尾加入内核要求内容，编辑完成通过“ESC”和“:wq”保存并退出编辑窗口
        ```
        kernel.shmall = 2097152
        kernel.shmmax = 2147483648
        kernel.shmmni = 4096
        kernel.sem = 250 32000 100 128
        net.ipv4.ip_local_port_range = 9000 65500
        net.core.rmem_default = 4194304
        net.core.rmem_max = 4194304
        net.core.wmem_default = 262144
        net.core.wmem_max = 1048586
        fs.file-max = 6815744
        ```
       - 使用`sysctl -p`生效新配置的系统内核参数
       ```
       [root@iZbp1itnf4brss8qb2rqhwZ ~]# sysctl -p
        fs.file-max = 1000000
        net.ipv4.tcp_max_tw_buckets = 6000
        net.ipv4.tcp_sack = 1
        net.ipv4.tcp_window_scaling = 1
        net.ipv4.tcp_rmem = 4096 87380 4194304
        net.ipv4.tcp_wmem = 4096 16384 4194304
        net.ipv4.tcp_max_syn_backlog = 16384
        net.core.netdev_max_backlog = 32768
        net.core.somaxconn = 32768
        net.core.wmem_default = 8388608
        net.core.rmem_default = 8388608
        net.core.rmem_max = 16777216
        net.core.wmem_max = 16777216
        net.ipv4.tcp_timestamps = 1
        net.ipv4.tcp_fin_timeout = 20
        net.ipv4.tcp_synack_retries = 2
        net.ipv4.tcp_syn_retries = 2
        net.ipv4.tcp_syncookies = 1
        net.ipv4.tcp_tw_reuse = 1
        net.ipv4.tcp_mem = 94500000 915000000 927000000
        net.ipv4.tcp_max_orphans = 3276800
        net.ipv4.ip_local_port_range = 1024 65000
        net.nf_conntrack_max = 6553500
        net.netfilter.nf_conntrack_max = 6553500
        net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
        net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
        net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
        net.netfilter.nf_conntrack_tcp_timeout_established = 3600
        kernel.shmall = 2097152
        kernel.shmmax = 2147483648
        kernel.shmmni = 4096
        kernel.sem = 250 32000 100 128
        net.ipv4.ip_local_port_range = 9000 65500
        net.core.rmem_default = 4194304
        net.core.rmem_max = 4194304
        net.core.wmem_default = 262144
        net.core.wmem_max = 1048586
        fs.file-max = 6815744
      ```
    - 配置Oracle用户shell limit<br>
      *为了提高linux系统运行性能，必须对Oracle用户进行以下限制*
      使用`vi /etc/security/limits.conf`命令打开配置文件，然后将下面的内容插入到文末
      ```
      oracle soft nproc 2047
      oracle hard nproc 16384
      oracle soft nofile 1024
      oracle hard nofile 65536
      oracle soft stack 10240
      oracle hard stack 10240
      ```
      注：noproc - 进程的最大数目<br>
      stack - 最大栈大小<br>
      nofile - 打开文件的最大数目<br>
      soft 指的是当前系统生效的设置值<br>
      hard 表明系统中所能设定的最大值<br>
      soft 的限制不能比har 限制高。用 - 就表明同时设置了 soft 和 hard 的值。<br>
      oracle：被限制的用户名，组名前面加@和用户名区别<br>
    - 编辑登录配置文件<br>
    *`vi /etc/pam.d/login `，然后进行登录配置文件的编辑，在文本最后添加：session required pam_limits.so或者session required /lib/security/pam_limits.so使shell limit生效。*
    ```
    session    required     pam_selinux.so open
    session    required     pam_namespace.so
    session    optional     pam_keyinit.so force revoke
    session    include      system-auth
    session    include      postlogin
    -session   optional     pam_ck_connector.so

    session required pam_limits.so
    ```
    - Oracle用户环境变量配置<br>
    *要成功安装并使用Oracle数据库软件，必须在Oracle用户的.bash_profile文件中设置ORACLE_BASE、ORACLE_HOME、ORACLE_SID和PATH环境变量，其他的根据需要来设置。ORACLE_HOME可以在安装前手动配置，另外，Oracle安装过程中会根据ORACLE_BASE的值自动指定的ORACLE_HOME，所以也可以在安装后将这个ORACLE_HOME写入.bash_profile。*




    

4. 
    