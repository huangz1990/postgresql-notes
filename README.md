# PostgreSQL笔记（持续更新中……！）



[TOC]



## 安装

用``apt``命令在Ubuntu上安装：

```bash
huangz@winpc:~$ sudo apt install postgresql
[sudo] password for huangz:
正在读取软件包列表... 完成
正在分析软件包的依赖关系树... 完成
正在读取状态信息... 完成
将会同时安装下列软件：
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "zh_CN.UTF-8".
The default database encoding has accordingly been set to "UTF8".
initdb: could not find suitable text search configuration for locale "zh_CN.UTF-8"
The default text search configuration will be set to "simple".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/14/main ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Asia/Shanghai
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
update-alternatives: 使用 /usr/share/postgresql/14/man/man1/postmaster.1.gz 来在自动模式中提供 /usr/share/man/man1/postmaster.1.gz (postmaster.1.gz)
正在设置 postgresql (14+238) ...
正在处理用于 man-db (2.10.2-1) 的触发器 ...
正在处理用于 libc-bin (2.35-0ubuntu3.1) 的触发器 ...
/sbin/ldconfig.real: /usr/lib/wsl/lib/libcuda.so.1 is not a symbolic link
```

注意安装过程中会出现一些配置信息，比如它会告诉你与本数据有关的文件都由用户``postgres`` 拥有，取得了用于存放数据库文件的``/var/lib/postgresql/14/main``文件夹的权限等等。



## 调用控制台

正如之前所言，用户账号默认没有权限：

```bash
huangz@winpc:~$ psql
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  role "huangz" does not exist
```

需要用PG本身的默认账号``postgres``才能连接：

```bash
huangz@winpc:~$ sudo -u postgres psql
could not change directory to "/home/huangz": 权限不够
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# help
You are using psql, the command-line interface to PostgreSQL.
Type:  \copyright for distribution terms
       \h for help with SQL commands
       \? for help with psql commands
       \g or terminate with semicolon to execute query
       \q to quit
```

默认账号本身是没有密码的，如果有需要的话，我们可以通过执行以下命令来为它设置密码：

```sql
ALTER USER postgres with encrypted password 'your_password';
```

更多信息可以参考Ubuntu的PG安装和配置指南：https://ubuntu.com/server/docs/databases-postgresql。
