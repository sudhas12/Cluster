### systemctl vs. init.d 
You are looking for the traditional init scripts in /etc/rc.d/init.d,
and they are gone? Here's an explanation on what's going on:

You are running a systemd-based OS where traditional init scripts have
been replaced by native systemd services files. Service files provide
very similar functionality to init scripts. To make use of service
files simply invoke "systemctl", which will output a list of all
currently running services (and other units). Use "systemctl
list-unit-files" to get a listing of all known unit files, including
stopped, disabled and masked ones. Use "systemctl start
foobar.service" and "systemctl stop foobar.service" to start or stop a
service, respectively. For further details, please refer to
systemctl(1).

Note that traditional init scripts continue to function on a systemd
system. An init script /etc/rc.d/init.d/foobar is implicitly mapped
into a service unit foobar.service during system initialization.

### Common Linux Commands  
##### checking memory and cpu on a machine
    /proc/cpuinfo
    /proc/meminfo
##### Adding a user
* sudo useradd \<username\>   
  * Example:Add user john
    * sudo useradd john

##### Setting/Changing user password    
* sudo passwd \<username\>   
 * Example:Set/change john's passwd to pwd   
   * sudo passwd john    

##### Add new group   
* groupadd \<groupname\>   
 * Example:Add group group1    
   * groupadd group1    

##### Add user to group   
* usermod -a -G \<groupname\> \<username\>   
 * Example:Add john to group1    
   * usermod -a -G group1 john  

##### Check a users group and other info about user 
id \<userid\>

##### Change a user's primary group. The lowercase 'g' option changes a primary group    
* usermod -g \<groupname\> \<username\>   
 * Example:Change john's primary group to group1     
   * usermod -g group1 john     

#####  View the Groups a User Account is Assigned To    
* groups   

##### Delete a group   
* groupdel \<groupname\>
 * Example:Delete group group1
   * groupdel group1

##### Delete a user
* userdel \<username\>
  * Example: Delete john
    * userdel john

##### Get list of users      
* sudo /etc/passwd

##### Get list of groups and users ina groups
* sudo cat /etc/groups | grep \<groupname\> 

#####Check users who have sudo privileges
* sudo cat /users/sudoers

##### Add the following Linux accounts to all nodes
* User `nimbus` with a UID of `2800`      
* User `igneous` with a UID of `2900`
* Command      
    useradd -u 2800  nimbus       
    useradd -u 2900  igneous

##### Add the following Linux accounts to all nodes
* Create the group `rocks` and add `igneous` to it    
* Create the group `clouds` and add `nimbus` to it   
* Command       
    groupadd rocks    
    groupadd clouds    
    usermod -a -G rocks igneous            
    usermod -a -G clouds nimbus  

##### List the `/etc/passwd` entries for `nimbus` and `igneous`
* Command cat /etc/passwd | grep -e nimbus -e igneous
* Result        
           nimbus\:\x\:2800\:2800\:\:/home/nimbus:/bin/bash      
           igneous\:\x\:2900\:2900\:\:/home/igneous:/bin/bash     

##### List the `/etc/group` entries for `rocks` and `clouds`    
* Command  cat /etc/group | grep -e rocks -e clouds
* Result     
           rocks\:\x\:2901\:igneous        
           clouds\:\x\:2902\:nimbus

##### List the Linux release you are using
* Command
 * cat /proc/version   - This will give the Kernel release number
 * Result  
    Linux version 2.6.32-642.3.1.el6.x86_64 (mockbuild@worker1.bsys.centos.org) (gcc version 4.4.7 20120313    
    (Red Hat 4.4.7-17) (GCC) ) #1 SMP Tue Jul 12 18:30:56 UTC 2016
 * Alternatively - This will give the kernel release number
 * Command   
   uname -a
 * Result  
   Linux quickstart.cloudera 2.6.32-573.el6.x86_64 #1 SMP Thu Jul 23 15:44:03 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 * This will give the distribution release number
 * Command   
   lsb_release -a   
   LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch   
   Distributor ID:	CentOS  
   Description:	CentOS release 6.7 (Final)   
   Release:	6.7   
   Codename:	Final  

##### Check clock sync
* Command    
     ntpstat
     ntpq -pn

#####  Verify nslookup <hostname> and nslookup <ipaddress>   

##### List the file system capacity for the first node
* Command
     df -H     
* Result    
        Filesystem      Size  Used Avail Use% Mounted on  
        /dev/sda1       106G  1.9G   99G   2% /   
        tmpfs           6.8G     0  6.8G   0% /dev/shm   
