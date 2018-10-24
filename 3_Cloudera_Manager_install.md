
### Install the JDBC Connector
1. Download & install JDBC connector 
* wget https://cdn.mysql.com/Downloads/Connector-J/mysql-connector-java-5.1.43.tar.gz 
* tar xvfz mysql-connector-java-5.1.43.tar.gz 
* mkdir /usr/share/java    
* Copy mysql-connector-java-5.1.43-bin.jar to another director with a DIFFERENT NAME (NOTE!!)   
* cp mysql-connector-java-5.1.43/mysql-connector-java-5.1.43-bin.jar /usr/share/java/mysql-connector-java.jar    

### Create databases & Grant privileges.        
* Log into mysql    
    mysql -u root -p <pwd>

* Create Databases and grant all to databases        
  create database <database_name> DEFAULT CHARACTER SET utf8;        
  grant all on \<database_name\>.* TO  username@'%' IDENTIFIED BY '\<password\>';      

* Repeat for all databases - scm,hue,hive,oozie,sentry,amon, nav, navms, rman. Best is to create a script     

* Check users have been created    
   SELECT User, Host, Password FROM mysql.user;

### Install Oracle jdk.
 *  Refer to documentation on how to install JDK: http://github.mtv.cloudera.com/ssubramaniam/install-bootcamp/blob/master/installation/labs/3A_Java_JDK8_Installation.md

### Cloundera Manager Installation on server
* Download the correct repo from: https://www.cloudera.com/documentation/enterprise/release-notes/topics/cm_vd.html     
 *  wget https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo
 *  mv cloudera-manager.repo /etc/yum.repos.d/   
 *  vi the file to change the baseurl to point to the specific version of Cloudera Manager you want to download.    
 *  Example, for Cloudera Manager version 5.9.3, change:    
    baseurl=http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/5.9.3/  
 *  yum repolist all | grep cloudera-manager
 *  yum update -y  cloudera-manager 
 *  yum install cloudera-manager-daemons cloudera-manager-server   

### Install the cloudera manager agent in each host   (this step can be skipped if CM installs the agent)
 *  wget https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo
 *  mv cloudera-manager.repo /etc/yum.repos.d/   
 *  vi the file to change the baseurl to point to the specific version of Cloudera Manager you want to download.   
 *  yum install cloudera-manager-agent cloudera-manager-daemons 
 
### Set $JAVA_HOME Directory and Configure Cloudera Manager to connect to the external database
* #Set JAVA_HOME to the directory where the JDK is installed 
* export JAVA_HOME=/usr/java/jdk.1.7.0_nn in the following files on Cloudera Manager Server and cluster hosts:   
* Making sure JAVE_HOME is named correctly to match what is in the scm_prepare_database.sh script is very important. 
* Also make sure the my-sql-connector jar name matches the name in the scm_prepare_database.sh script.
* Eg:   
*     

     export JAVA_HOME=/usr/java/jdk1.7
    #
    * Locate the JDBC driver jar file.

    export CMF_JDBC_DRIVER_JAR="/usr/share/java/mysql-connector-java-5.1.43-bin.jar:/usr/share/java/oracle-connector-java.jar"

 * Cloudera Manager Server - /etc/default/cloudera-scm-server. This change affects only the Cloudera Manager Server process, and does not affect the Cloudera Management Service roles.

* Verify/Edit the following script:    
vi /etc/cloudera-scm-server/db.properties   

* The database type: Currently 'mysql', 'postgresql' and 'oracle' are valid databases.   
   com.cloudera.cmf.db.type=mysql   

* The database host. If a non standard port is needed, use 'hostname:port'    
  com.cloudera.cmf.db.host=<hostname> eg: sub-1.gce.cloudera.com

* The database name   
  com.cloudera.cmf.db.name=scm   

* The database user   
  com.cloudera.cmf.db.user=scm  

* The database user's password     
  com.cloudera.cmf.db.password=scm   

* The db setup type 
* By default, it is set to INIT   
* If scm-server uses Embedded DB then it is set to EMBEDDED   
* If scm-server uses External DB then it is set to EXTERNAL  
   com.cloudera.cmf.db.setupType=EXTERNAL   

* Run the following script    
* /usr/share/cmf/schema/scm_prepare_database.sh database-type [options] database-name username password
  /usr/share/cmf/schema/scm_prepare_database.sh mysql  scm scm scm    (if this fails look for error sin log @/var/log/cloudera-scm-server

* Start Cloudera Manager server service 
 *  service cloudera-scm-server start
 
 * VERIFY - CM runs as a Java process
 
 1.Cloudera	Manager	Server	runs	as	a	java	process	that you	can	view	by	using	the	
top Linux	utility.	Notice	the	CPU	usage	is	in	the	90th percentile or	above for	a	
few	seconds	while	the	server	starts.	Once	the	CPU	usage	drops, the	Cloudera	
Manager	browser	interface	will	become available.

    top
 2. ps wax | grep cloudera-scm-server
 
 The	results	of	the	ps command	above	show	that	Cloudera	Manager	Server	is	
using	the	JDBC	MySQL	connector	to	connect	to	MySQL.	It	also	shows	logging	
configuration	and	other	details.


### Creating and Using a Parcel Repository for Cloudera Manager

 * Refer to https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_create_local_parcel_repo.html#concept_cdc_kbk_mz. 
 * Includes instructions for
    * Hosting a Parcel Repository
        * Installing Apache HTTPD
        * Starting httpd service
        * Downloading the Parcel and Publishing Files
    * Configuring the Cloudera Manager Server to Use the Parcel URL for Hosted Repositories
    * Using a Local Parcel Repository
    









