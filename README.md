# psql
psql note

# 安装
## 这里主要介绍从源码安装
### 1.从官网下载软件包，这里以9.1.4为例，https://ftp.postgresql.org/pub/source/v9.1.4/postgresql-9.1.4.tar.gz 然后，tar -xzvf postgresql-9.1.4.tar.gz
### 2.三板斧，./configure
./configure --prefix=/home/tensorflow/pgsql9.1.4 --with-perl --with-python
--with*是为了扩展可用编程语言
建立软连接方便后面软件更新：ln -sf /home/tensorflow/pgsql9.1.4/ /home/tensorflow/pgsql
### 3.make
### 4.make install

## 安装后的配置
### 1.设置可执行路径: export PATH=/home/tensorflow/pgsql/bin:$PATH
### 2.设置共享库的路径: export LD_LIBRARY_PATH=/home/tensorflow/pgsql/lib

## 创建数据库簇（这步是干啥？）
### 1.先设定数据库中数据目录的环境变量：export PGDATA=/home/tensorflow/osdba/pgdata
### 2.执行下面的命令创建数据库簇：initdb

## 安装contrib目录下工具
### cd postgresql-9.1.4/contrib | make | sudo make install

## 启动数据库：pg_ctl start -D $PGDATA
## 停止数据库： pg_ctl stop -D $PGDATA -m fast
### -m 后面有三种关闭模式，smart-等所有连接终止后关闭，fast-快速关闭，已有事物回滚，immediate-立即关闭，下次需要恢复
