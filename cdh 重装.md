cdh 重装

# 停止CM Server 和 所有Agent

```
/opt/cloudera-manager/cm-5.16.1/etc/init.d/cloudera-scm-agent stop
/opt/cloudera-manager/cm-5.16.1/etc/init.d/cloudera-scm-server stop

#卸载服务
for i in $(rpm -qa|grep cloudera);do rpm -e $i --nodeps ;done
```

# 删除服务数据

```
# 我这里只按安装了hdfs、yarn、zookeeper三个服务，所以只删除这三个目录。具体的数据存放位置，需要在CDH提供的web界面上查看，并记录下载，留待下面删除
rm -rf /dfs /yarn /var/lib/zookeeper /var/log/zookeeper
```

# 在所有节点上杀死所有 cloudera 运行的进程

```
kill -9 $(pgrep -f cloudera)
```

# 卸载CDH的磁盘挂载

```
df -h
umount /opt/cloudera-manager/cm-5.16.1/run/cloudera-scm-agent/process
# 若 umount 卸载时，被夯住了，则执行下面命令
yum install -y lsof
kill -9 $(lsof /opt/cloudera-manager/cm-5.12.0/run/cloudera-scm-agent/process)
```



# 删除CDH服务组件的配置文件及软件安装目录

```
rm -rf /opt/cloudera
rm -rf /opt/cloudera-manager
rm -rf /var/lib/cloudera*
rm -rf /var/log/cloudera*
rm -rf /run/cloudera-scm-agent/
rm -rf /etc/security/limits.d/cloudera-scm.conf
rm -rf /etc/cloudera
rm -rf /etc/hadoop*
rm -rf /etc/zookeeper
rm -rf /etc/hive*
rm -rf /etc/hbase*
rm -rf /etc/solr
rm -rf /etc/impala
rm -rf /etc/spark
rm -rf /etc/sqoop*
rm -rf /tmp/scm_*
rm -rf /tmp/.scm_*
rm -rf /tmp/hsperfdata_cloudera-scm
rm -rf /var/local/kafka/
# 删除所有系统中所有匹配 cloudera 字符串的文件，可不用执行。因为可能删掉文件名带有cloudera字符串，实际与CDH无关系的文件
find / -name '*cloudera*' | while read line; do rm -rf ${line}; done
```





# 删除数据库

```
drop database amon;
drop database cmf;
```

# 删除 cloudera-scm 用户

```
userdel cloudera-scm
```







node-1 上面的mysql   密码：Zhujie0814.



在MYSQL中创建需要用到的数据库：



```php
mysql> CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
mysql> CREATE DATABASE hive DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
mysql> CREATE DATABASE rmon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
mysql> CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci; 
mysql> CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```



在MYSQL中创建集群所用的统一登陆用户授权：



```bash
mysql> CREATE USER 'cdh'@'%' IDENTIFIED BY 'Zhujie0814.'; 
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL ON *.* TO 'cdh'@'%';
Query OK, 0 rows affected (0.00 sec)
```


vim /etc/cloudera-scm-agent/config.ini


./scm_prepare_database.sh -h node-1 -P 3306 mysql scm cdh Zhujie0814.



systemctl start cloudera-scm-agent


安装CDH6.2.1，在使用systemctl start cloudera-scm-server启动，status查看时，启动失败。
查看/var/log/cloudera-scm-server 下没有日志
再查看服务相关的日志： journalctl -xe 
