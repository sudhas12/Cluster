
##<center>Install and Configure MySQL with a replica server including mysql client and JDK 
### Install SQL-Server and SQL-Client
1. Download the rpm  
 * wget http://repo.mysql.com/mysql-community-release-el6.rpm  
 
2a. Install the rpm  
 * rpm -ivh mysql-community-release-el6.rpm 

2b. Verify rpm was installed
 * rpm -qa | grep ysql
 
3. Update/refresh the yum repo 
 * yum update -y mysql-5.1.73-8.el6_8.x86_64   
   mysql-libs-5.1.73-8.el6_8.x86_64
 
4. Check which versions are enabled   
 * yum repolist all | grep mysql  
 
   mysql-connectors-community        MySQL Connectors Community     enabled:     39  
   mysql-connectors-community-source MySQL Connectors Community - S disabled   
   mysql-tools-community             MySQL Tools Community          enabled:     49   
   mysql-tools-community-source      MySQL Tools Community - Source disabled   
   mysql55-community-source          MySQL 5.5 Community Server - S disabled   
   mysql56-community                 MySQL 5.6 Community Server     enabled:    377   
   mysql56-community-source          MySQL 5.6 Community Server - S disabled   
   mysql57-community                 MySQL 5.7 Community Server     disabled   
   mysql57-community-source          MySQL 5.7 Community Server - S disabled    

5. Disable mysql version 6 and enable version 5 
 * yum-config-manager --disable mysql56-community 
 * yum-config-manager --enable mysql55-community 

   mysql55-community                 MySQL 5.5 Community Server     disabled   
6. ######Altenatively use vi to enable version 5 and disable version 5.  
   *   vi /etc/yum.repos.d/mysql-community.repo 
   *   Set enabled=1 for version 5.5 and set enabled=0 for the remaining versions 

7. Verify that only version 5.5 is enabled  
 * yum repolist all | grep mysql 

8. Install mysql-server AND mysql (client) on node 1 and node 2
 * yum install mysql-server mysql -y  (or yum install mysql-community-server)

9. Install just mysql (client) on remaining nodes 
 * yum install mysql -y (sudo yum install mysql-community)

10. Start mysql service
 * service mysqld start
 * NOTE:  If there are errors related to tables and if the error msg indicates that mysql_upgrade needs to be run, then run it while mysqld service is running. Execute command not from within mysql but as a shell command
 * mysql_upgrade
 * service mysqld restart 
 
 11. Following steps below for configuring a HA server on first master node  
 *  (Add following lines after [mysqld)  

       server-id = 1    
       binlog-do-db=newdatabase    
       binlog-do-db=amon   
       binlog-do-db=rman        
       binlog-do-db=hive       
       binlog-do-db=scm        
       binlog-do-db=hue        
       binlog-do-db=oozie      
       binlog-do-db=amon     
       binlog-do-db=nav     
       binlog-do-db=navms     
       binlog-do-db=sentry     
       binlog_format=row       
       relay-log = /var/lib/mysql/mysql-relay-bin    
       relay-log-index = /var/lib/mysql/mysql-relay-bin.index     
       log-error = /var/lib/mysql/mysql.err     
       master-info-file = /var/lib/mysql/mysql-master.info    
       relay-log-info-file = /var/lib/mysql/mysql-relay-log.info        
       log-bin = /var/lib/mysql/mysql-bin    
      
12.vi /etc/my.cnf  on first replica server   
  *  (Add following lines after [mysqld] 

    server-id = 2    
    binlog-do-db=newdatabase     
    binlog-do-db=amon     
    binlog-do-db=rman     
    binlog-do-db=metastore     
    binlog-do-db=scm     
    binlog-do-db=hue     
    binlog-do-db=oozie    
    binlog_format=row     
    relay-log = /var/lib/mysql/mysql-relay-bin    
    relay-log-index = /var/lib/mysql/mysql-relay-bin.index    
    log-error = /var/lib/mysql/mysql.err   
    master-info-file = /var/lib/mysql/mysql-master.info     

13. Restart mysql service     
 * service mysqld restart    

14. Create a new user     
 *    CREATE USER 'slave-user'@'%' IDENTIFIED BY 'slave-password';       

15. execute following cmd within mysql   
 *  mysql> GRANT REPLICATION SLAVE ON *.* TO 'slaveuser'@'%' IDENTIFIED BY 'cloudera';      
 *  mysql> FLUSH PRIVILEGES;     
 *  mysql> FLUSH TABLES WITH READ LOCK; 
 *  mysql> SHOW MASTER STATUS; 
 *  mysql >UNLOCK TABLES; 
 *  mysql > QUIT; 

16. show master status; 

+------------------+----------+-----------------------------------------------+------------------+-------------------+  
| File             | Position | Binlog_Do_DB                                  | Binlog_Ignore_DB | Executed_Gtid_Set |  
+------------------+----------+-----------------------------------------------+------------------+-------------------+  
| mysql-bin.000001 |      402 | newdatabase,amon,rman,metastore,scm,hue,oozie |                  |                   |  
+------------------+----------+-----------------------------------------------+------------------+-------------------+  
1 row in set (0.00 sec) 

17. Go mysql on slave node and run following 
 * mysql >CHANGE MASTER TO MASTER_HOST='172.31.113.198',MASTER_USER='slave_user', MASTER_PASSWORD='your_password',      
 * MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=403;        
 * mysql> start SLAVE     

17. Go to master node mysql and run followinging 
 * mysql> create database newdatabase; 
 * mysql> use newdatabase; 
 * mysql> create table test(a int); 
 * mysql> insert into test values(2); 
 * mysql> select * from test; 
+------+     
| a    |    
+------+     
|    2 |     
+------+    
1 row in set (0.00 sec) 

mysql> show databases;     
+--------------------+   
| Database           |   
+--------------------+   
| information_schema |   
| mysql              |   
| newdatabase        |   
| performance_schema |    
| replicationtest    |     
+--------------------+      
5 rows in set (0.00 sec)     

18. Go to replication node and run following:    
 * mysql> show slave status\G 
*************************** 1. row *************************** 
               Slave_IO_State: Waiting for master to send event 
                  Master_Host: 172.31.117.210 
                  Master_User: slaveuser 
                  Master_Port: 3306 
                Connect_Retry: 60 
              Master_Log_File: mysql-bin.000001 
          Read_Master_Log_Pos: 402 
               Relay_Log_File: mysql-relay-bin.000002 
                Relay_Log_Pos: 283 
        Relay_Master_Log_File: mysql-bin.000001 
             Slave_IO_Running: Yes 
            Slave_SQL_Running: Yes 
              Replicate_Do_DB:  
          Replicate_Ignore_DB:  
           Replicate_Do_Table:  
       Replicate_Ignore_Table:  
      Replicate_Wild_Do_Table:  
  Replicate_Wild_Ignore_Table:  
                   Last_Errno: 0 
                   Last_Error:  
                 Skip_Counter: 0 
          Exec_Master_Log_Pos: 402 
              Relay_Log_Space: 456 
              Until_Condition: None 
               Until_Log_File:  
                Until_Log_Pos: 0 
           Master_SSL_Allowed: No 
           Master_SSL_CA_File:  
           Master_SSL_CA_Path:  
              Master_SSL_Cert:  
            Master_SSL_Cipher:  
               Master_SSL_Key:  
        Seconds_Behind_Master: 0 
Master_SSL_Verify_Server_Cert: No 
                Last_IO_Errno: 0 
                Last_IO_Error:  
               Last_SQL_Errno: 0 
               Last_SQL_Error:  
  Replicate_Ignore_Server_Ids:  
             Master_Server_Id: 1 
                  Master_UUID: aeb6ce6b-7d59-11e7-8994-4201ac1f75d2 
             Master_Info_File: /var/lib/mysql/mysql-master.info 
                    SQL_Delay: 0 
          SQL_Remaining_Delay: NULL 
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it 
           Master_Retry_Count: 86400 
                  Master_Bind:  
      Last_IO_Error_Timestamp:  
     Last_SQL_Error_Timestamp:  
               Master_SSL_Crl:  
           Master_SSL_Crlpath:  
           Retrieved_Gtid_Set:  
            Executed_Gtid_Set:  
                Auto_Position: 0 
1 row in set (0.00 sec) 

 * mysql> use newdatabase;     
   Reading table information for completion of table and column names 
   You can turn off this feature to get a quicker startup with -A 

    Database changed     
 * mysql> select * from test;    
     +------+    
     | a    |    
     +------+     
     |    2 |     
     +------+    
1 row in set (0.00 sec)   
 
 * mysql> quit; 

17. Secure MySQL installation using  
 *    /usr/bin/mysql_secure_installation      

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL 
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY! 
 
In order to log into MySQL to secure it, we'll need the current 
password for the root user.  If you've just installed MySQL, and  
you haven't set the root password yet, the password will be blank, 
so you should just press enter here. 

Enter current password for root (enter for none):  
OK, successfully used password, moving on... 

Setting the root password ensures that nobody can log into the MySQL 
root user without the proper authorisation. 

Set root password? [Y/n] y 
New password:  
Re-enter new password:  
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment. 

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
ERROR 1008 (HY000) at line 1: Can't drop database 'test'; database doesn't exist
 ... Failed!  Not critical, keep moving...
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!






