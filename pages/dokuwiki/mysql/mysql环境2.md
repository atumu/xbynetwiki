title: mysql环境2 

#  MySQL之my.ini配置 
http://www.linuxidc.com/Linux/2013-09/90110.htm
http://linux.cn/article-3912-1-weixin.html
系统默认是按/etc/my.cnf-----/etc/mysql/my.cnf----/usr/local/mysql/my.cnf的顺序读取配置文件的，当有多个配置文件时，mysql会以读取到的最后一个配置文件中的参数为准。
常用的配置参数有：
**default-character-set = utf8** 
**basedir = /opt/mysql
datadir = /opt/mysql/var**
**port            = 3306** mysqld服务运行时的端口号，默认为3306
**socket          = /tmp/mysql.sock** socket文件是linux/unix系统特有的，用户在该环境下的客户端连接可以不通过tcp/ip网络，而直接使用socket文件连接
back_log  = 300 该值为设定档mysql暂时停止响应新的请求前，短时间内有多少个请求可以存在堆栈内，**如果系统在短时间内有很多的连接，应该增大该值，该值最好设置小于512的整数**
skip-networking 不在tcp/ip端口上进行监听，所有的连接都是通过**本地的socket文件**连接，**这样可以提高安全性，确定是不能通过网络连接数据库。**
skip-locking 避免mysql的外部锁定，增强稳定性
skip-name-resolve 避免mysql对外部的连接进行DNS解析，**若使用此设置，那么远程主机连接时只能使用ip，而不能使用域名**
**max_connections = 3000 指定mysql服务所允许的最大连接数，**
max_connect_errors = 1000 每个主机连接允许异常中断的次数，当超过该次数mysql服务器将禁止该主机的连接请求，直到mysql服务重启，或者flushhosts命令清空host的相关信息
table_cache = 614k 表的高速缓冲区的大小，当mysql访问一个表时，如果mysql表缓冲区还有空间，那么这个表就会被打开通放入高速缓冲区，好处是可以更快速的访问表中的内容。
如果open_tables和opened_tables的值接近该值，那么久该增加该值的大小
**max_allowed_packet = 4M** **设定在网络传输中一次可以传输消息的最大值，系统默认为1M，**最大可以是1G
**sort_buffer_size = 16M** **排序缓冲区用来处理类似orderby以及groupby队列所引起的排序，系统默认大小为2M**,该参数对应分配内存是每个连接独占的，若有100个连接，实际分配的排序缓冲区大小为6*100；推荐设置为6M-8M
**join_buffer_size=8M** 联合查询操作所使用的缓冲区大小。
**thread_cache_size = 64** 设置threadcache池中**可以缓存连接线程的最大数量，默认为0，该值表示可以重新利用保存在缓存中线程的数量，当断开连接时若缓存中还有空间，那么客户端的线程将被放到缓存中，如果线程重新被请求，那么请求将从缓存中读取，若果缓存中是空的或者是新的请求，那么线程将被重新创建。设置规律为：1G内存设置为8,2G内存设置为16,4G以上设置为64**
query_cache_size = 64M 指定mysql查询缓冲区的大小，用来缓冲select的结果，并在下一次同样查询的时候不再执行查询而直接返回结果，根据Qcache_lowmem_prunes的大小，来查看当前的负载是否足够高
query_cache_limit = 4M 只有小于该值的结果才被缓冲，放置一个极大的结果将其他所有的查询结果都覆盖
tmp_table_size 256M 内存临时表的大小，如果超过该值，会将临时表写入磁盘
**default_storage_engine = MYISAM 创建表时默认使用的存储引擎**

**log-bin=mysql-bin 打开二进制日志功能**
key_buffer_size = 384M **指定索引缓冲区的大小**，内存为4G时刻设置为256M或384M
read_buffer_size = 8M 用来做MYISAM表全表扫描的缓冲大小

##  MySQL配置文件my.cnf 常用配置实例 
[client] 
default-character-set = utf8 
port = 3306 
socket = /tmp/mysql.sock 
[mysqld] 
user = mysql 
port = 3306 
socket = /tmp/mysql.sock 
basedir = /opt/mysql
datadir = /opt/mysql/var
log-error = /opt/mysql/var/mysql-error.log 
pid-file = /opt/mysql/var/mysql.pid 
log_slave_updates = 1 
log-bin = /opt/mysql/var/mysql-bin
binlog_format = mixed 
binlog_cache_size = 4M 
max_binlog_cache_size = 8M 
max_binlog_size = 1G 
expire_logs_days = 90 
key_buffer_size = 384M 
sort_buffer_size = 2M 
read_buffer_size = 2M 
read_rnd_buffer_size = 16M 
join_buffer_size = 2M 
thread_cache_size = 8 
query_cache_size = 32M 
query_cache_limit = 2M 
query_cache_min_res_unit = 2k 
thread_concurrency = 32 
table_cache = 614 
table_open_cache = 512 
open_files_limit = 10240 
back_log = 600 
max_connections = 5000 
max_connect_errors = 6000 
external-locking = FALSE 
max_allowed_packet = 16M 
default-storage-engine = MYISAM 
thread_stack = 192k 
transaction_isolation = READ-COMMITTED 
tmp_table_size = 256M 
max_heap_table_size = 512M 
bulk_insert_buffer_size = 64M 
myisam_sort_buffer_size = 64M 
myisam_max_sort_file_size = 10G 
myisam_repair_threads = 1 
myisam_recover 
long_query_time = 2 
slow_query_log 
slow_query_log_file = /opt/mysql/var/slow.log 
skip-name-resolve 
skip-locking 
skip-networking 
innodb_additional_mem_pool_size = 16M 
innodb_buffer_pool_size = 512M 
innodb_data_file_path = ibdata1:256M:autoextend 
innodb_file_io_threads = 4 
innodb_thread_aoncurrency = 8 
innodb_flush_log_at_trx_commit = 2 
innodb_log_buffer_size = 16M 
innodb_log_file_size = 128M 
innodb_log_files_in_group = 3 
innodb_max_dirty_pages_pct = 90 
innodb_lock_wait_timeout = 120 
innodb_file_per_table = 0 
[mysqldump] 
quick 
max_allow_packet = 64M 
[mysql] 
no-auto-rehash 
safr-updates 
[myisamchk] 
key_buffer_size = 256M 
sort_buffer_size = 256M 
read_buffer = 2M 
[mysqldump] 
quick 
max_allow_packet = 64M 
[mysql] 
no-auto-rehash 
safe-updates 
[myisamchk] 
key_buffer_size = 256M 
sort_buffer_size = 256M 
read_buffer = 2M 
write_buffer = 2M 
[mysqlhotcopy] 
interactive-timeout