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



## 编译&测试

PG的源码可以在这里找到：[git.postgresql.org Git - postgresql.git/summary](https://git.postgresql.org/gitweb/?p=postgresql.git)

编译步骤可以在这里找到：[PostgreSQL: Documentation: 15: Chapter 17. Installation from Source Code](https://www.postgresql.org/docs/current/installation.html)

根据[Compile and Install from source code - PostgreSQL wiki](https://wiki.postgresql.org/wiki/Compile_and_Install_from_source_code)，编译需要安装以下一些库：

```bash
$ sudo apt-get install build-essential libreadline-dev zlib1g-dev flex bison libxml2-dev libxslt-dev libssl-dev libxml2-utils xsltproc ccache
```

获取源码：

```bash
$ git clone git://git.postgresql.org/git/postgresql.git
```

配置：

```bash
$ ./configure --without-icu
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
...
config.status: linking src/backend/port/sysv_shmem.c to src/backend/port/pg_shmem.c
config.status: linking src/include/port/linux.h to src/include/pg_config_os.h
config.status: linking src/makefiles/Makefile.linux to src/Makefile.port
```

> TODO
>
> 安装了icu库但是没有找到，所以这里编译的时候关闭了该库，待解决……
>
> [ICU - International Components for Unicode](https://icu.unicode.org/)
>
> [libicu-dev_67.1-7_amd64.deb Debian 11 Download (pkgs.org)](https://debian.pkgs.org/11/debian-main-amd64/libicu-dev_67.1-7_amd64.deb.html)

编译：

```bash
$ make
make -C ./src/backend generated-headers
make[1]: Entering directory '/home/huangz/postgresql/src/backend'
...
make[1]: Entering directory '/home/huangz/postgresql/config'
make[1]: Nothing to be done for 'all'.
make[1]: Leaving directory '/home/huangz/postgresql/config'
```

测试：

```bash
$ make check
...
# +++ regress check in src/test/regress +++
# using temp instance on port 61696 with PID 51090
ok 1         - test_setup                                106 ms
# parallel group (20 tests):  varchar char name oid int2 text txid pg_lsn boolean int4 uuid float4 regproc money int8 float8 bit enum numeric rangetypes
ok 2         + boolean                                    19 ms
ok 3         + char                                        9 ms
ok 4         + name                                       11 ms
ok 5         + varchar                                     8 ms
...
ok 214       - fast_default                               56 ms
ok 215       - tablespace                                128 ms
1..215
# All 215 tests passed.
make[1]: Leaving directory '/home/huangz/postgresql/src/test/regress'
```



## 创建/删除数据库

创建数据库需要用到``CREATE DATABSE``命令：

```sql
CREATE DATABSE name;
```

执行以下命令以创建``testdb``：

```sql
postgres=# CREATE DATABASE testdb;
CREATE DATABASE
```

该命令的详细信息请见：[PostgreSQL: Documentation: devel: 23.2. Creating a Database](https://www.postgresql.org/docs/devel/manage-ag-createdb.html)

删除数据库可以使用``DROP DATABASE`` ：

```sql
testdb=# CREATE DATABASE anotherdb;
CREATE DATABASE
testdb=# DROP DATABASE anotherdb;
DROP DATABASE
```

更多信息[PostgreSQL: Documentation: devel: DROP TABLE](https://www.postgresql.org/docs/devel/sql-droptable.html)。

查看数据库使用``\l``：

```sql
testdb=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 |
 template0 | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 |
(4 rows)
```



## 列出数据库&切换数据库

使用``\l``或者``\list``可以列出当前PG服务器拥有的数据库：

```sql
postgres=# \list
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 |
 template0 | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 |
(4 rows)
```

使用``\c``或者``\connect``可以切换至指定的数据库：

```sql
postgres=# \connect testdb
You are now connected to database "testdb" as user "postgres".
```

这种以反斜杠开头的命令被``psql``称为元命令，更多元命令用法可见：[PostgreSQL: Documentation: devel: psql](https://www.postgresql.org/docs/devel/app-psql.html)。



## 创建/删除表

创建表需要用到``CREATE TABLE``命令：

```sql
testdb=# CREATE TABLE person (
testdb(#   id SERIAL PRIMARY KEY,
testdb(#   name VARCHAR(80),
testdb(#   age SMALLINT
testdb(# );
CREATE TABLE
```

之后用``\dt``命令可以查看被创建的表：

```sql
testdb=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner
--------+--------+-------+----------
 public | person | table | postgres
(1 row)
```

创建表的更多信息可以参考[PostgreSQL: Documentation: devel: createdb](https://www.postgresql.org/docs/devel/app-createdb.html)。

当不再需要某个表的时候，可以使用``DROP TABLE``将其删除。作为例子，以下代码先创建然后再删除了``dummy``表：

```sql
testdb=# CREATE TABLE dummy ();
CREATE TABLE
testdb=# DROP TABLE dummy;
DROP TABLE
```

``DROP TABLE``的更多信息可见：[PostgreSQL: Documentation: devel: DROP TABLE](https://www.postgresql.org/docs/devel/sql-droptable.html)。



## 

