# 01.oracle-test
oracle-learning




一、Oracle安装环境准备
1.配置yum源，安装相关包
rpm -ivh binutils-2.23.52.0.1-55.el7.x86_64.rpm
#rpm -ivh compat-libstdc++-33-3.2.3
rpm -ivh elfutils-libelf-0.163-3.el7.x86_64.rpm
rpm -ivh elfutils-libelf-devel-0.163-3.el7.x86_64.rpm
#rpm -ivh elfutils-libelf-devel-static-0.125
rpm -ivh gcc-4.8.5-4.el7.x86_64.rpm
rpm -ivh gcc-c++-4.8.5-4.el7.x86_64.rpm
rpm -ivh glibc-2.17-105.el7.x86_64.rpm
rpm -ivh glibc-common-2.17-105.el7.x86_64.rpm
rpm -ivh glibc-devel-2.17-105.el7.x86_64.rpm
rpm -ivh glibc-headers-2.17-105.el7.x86_64.rpm
rpm -ivh kernel-headers-3.10.0-327.el7.x86_64.rpm
rpm -ivh ksh-20060214
rpm -ivh libaio-0.3.109-13.el7.x86_64.rpm
rpm -ivh libaio-devel-0.3.109-13.el7.x86_64.rpm
rpm -ivh libgcc-4.8.5-4.el7.x86_64.rpm 
rpm -ivh libgomp-4.8.5-4.el7.x86_64.rpm
rpm -ivh libstdc++-4.8.5-4.el7.x86_64.rpm
rpm -ivh libstdc++-devel-4.8.5-4.el7.x86_64.rpm
rpm -ivh make-3.82-21.el7.x86_64.rpm
rpm -ivh sysstat-10.1.5-7.el7.x86_64.rpm
rpm -ivh unixODBC-2.3.1-11.el7.x86_64.rpm 
rpm -ivh unixODBC-devel-2.3.1-11.el7.x86_64.rpm 
2.配置内核参数
a.修改/etc/sysctl.conf
vim /etc/sysctl.conf
添加以下内容
# for oracle
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 4101769216
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
配置生效 
#/sbin/sysctl -p  
b.修改/etc/security/limits.conf
vim /etc/security/limits.conf
添加以下内容
# for oracle
oracle              soft    nproc   2047
oracle              hard    nproc   16384
oracle              soft    nofile  1024
oracle              hard    nofile  65536
oracle              soft    stack   10240

3.创建Oracle用户和用户组
groupadd oinstall
groupadd dba
groupadd oper
useradd -g oinstall -G dba oracle
passwd oracle
本次配置密码为000000，根据需要自定义

4.创建Oracle Base目录:
mkdir -p /u01/app/
chown -R oracle:oinstall /u01/app/
chmod -R 775 /u01/app/


5.编辑Oracle用户环境，编辑.bash_profile文件
# su - oracle
$ vi .bash_profile
编辑.bash_profile文件，添加以下内容
# For Oracle
export DISPLAY=:0.0
export TMP=/tmp;
export TMPDIR=$TMP;
export ORACLE_BASE=/u01/app/oracle;
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1;
export ORACLE_SID=sales;
export ORACLE_TERM=xterm;
export PATH=/usr/sbin:$PATH;
export PATH=$ORACLE_HOME/bin:$PATH;
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib;
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib;


二.安装Oracle软件与数据库
1.安装Oracle软件(11.2.0.4版本)
上传Oracle的安装文件p13390677_112040_Linux-x86-64_1of7.zip与p13390677_112040_Linux-x86-64_2of7.zip并解压缩.
以oracle用户身份登录系统安装Oracle，为避免出现中文乱码，装装前可以执行export LANG=""，显示英文，
$ export LANG=""
$ cd database
$ ./runInstaller
在安装时选择只安装Oracle软件。

安装最后步骤按提示在root用户下执行脚本.
# /u01/app/oraInventory/orainstRoot.sh
# /u01/app/oracle/product/11.2.0/db_1/root.sh


2.安装数据库
Oracle软件安装完后，需要执行命令netca配置监听器.
$ netca
在图形界面中按提示配置监听器.
然后执行命令dbca安装数据库.
$ dbca
在图形界面中按提示安装数据库就可以了。

四.测试运行安装的Oracle系统
数据库安装完后监听器与数据库实例就已启动。执行以下测试监听器与实例.
$ lsnrctl stop
$ lsnrctl start
$ sqlplus /nolog
SQL> connect / as sysdba;
SQL> shutdown
SQL> startup
执行其它SQL语句测试数据库.




三、问题处理
1.解决ORA-01078 & LRM-00109 
问题现象：
SQL> 
SQL> startup 
ORA-01078: failure in processing system parameters
LRM-00109: could not open parameter file '/u01/app/oracle/product/11.2.0/db_1/dbs/initsales.ora'
问题原因：
数据库默认将使用spfile启动数据库，如果spfile不存在，则就会出现上述错误
解决方案：
将$ORACLE_BASE/admin/数据库名称/pfile目录下的init.ora.012009233838形式的文件copy到$ORACLE_HOME/dbs目录下initoracle.ora即可。

2.解决ORA-01102: cannot mount database in EXCLUSIVE mode问题
问题现象：
SQL> startup;
ORACLE instance started.
Total System Global Area 2455228416 bytes
Fixed Size		    2255712 bytes
Variable Size		  603980960 bytes
Database Buffers	 1845493760 bytes
Redo Buffers		    3497984 bytes
ORA-01102: cannot mount database in EXCLUSIVE mode
问题原因：
lk<SID>文件被占用造成的，该文件位于ORALCE_HOME下的dbs目录下，使用fuser查看占用状态并释放即可。
解决方案：
[oracle@orcl dbs]$ pwd
/u01/app/oracle/product/11.2.0/db_1/dbs
[oracle@orcl dbs]$ fuser -u lkORCL 
/u01/app/oracle/product/11.2.0/db_1/dbs/lkORCL: 23251(oracle) 23253(oracle) 23259(oracle) 23263(oracle) 23267(oracle) 23269(oracle) 23271(oracle) 23273(oracle) 23275(oracle) 23277(oracle) 23279(oracle) 23281(oracle)
[oracle@orcl dbs]$ fuser -k lkORCL 
/u01/app/oracle/product/11.2.0/db_1/dbs/lkORCL: 23251 23253 23259 23263 23267 23269 23271 23273 23275 23277 23279 23281
[oracle@orcl dbs]$ fuser -u lkORCL 




四、数据库操作
数据库操作：
SQL> select tablespace_name,file_id,bytes/1024/1024,file_name from dba_data_files order by file_id;
SQL> select table_name from user_tables;
select inspur from tab/dba_tables/dba_objects/cat; 
select * from inspur;
shutdown immediate;

1.创建数据表空间、索引表空间
/*第1步：创建数据表空间  */
SQL> create tablespace dbdata  
logging  
datafile '/u01/app/oradata/o6/dbdata.dbf' 
size 50m  
autoextend on  
next 50m maxsize 20480m  
extent management local;  
 
/*第2步：创建索引表空间  */
SQL> create tablespace indexdb  
logging  
datafile '/u01/app/oradata/o6/indexdb.dbf' 
size 50m  
autoextend on  
next 50m maxsize 20480m  
extent management local;  
/*第3步：检查插入是否成功 */
SQL> select tablespace_name,file_id,bytes/1024/1024,file_name from dba_data_files order by file_id;


2.插入，查找，修改
/*1.创建表inspur */
SQL> create table inspur
(
id char(5),
name varchar(15),
tel char(10)
);
/*2.表中插入rows */
SQL> INSERT INTO inspur  (id, name, tel) VALUES ( 1, 11, 111);
SQL> INSERT INTO inspur  (id, name, tel) VALUES ( 2, 22, 222);
SQL> INSERT INTO inspur  (id, name, tel) VALUES ( 3, 33, 333);
SQL> INSERT INTO inspur  (id, name, tel) VALUES ( 4, 44, 444);
/*3.修改表 */
#重命名表
SQL> ALTER TABLE inspur RENAME TO inspur1;
#重命名列
SQL> ALTER TABLE inspur RENAME COLUMN tel to telephone;
/*4.commit提交修改 */
SQL> commit；



