### 1.  Check vm.swappiness on all your nodes.  

    Virtual Memory swapping can have a large impact on the performance of a Hadoop system. Because of the memory requirements of YARN containers and processes running on the nodes in a cluster, swapping process out of memory to disk can cause serious performance limitations. As such, the historical recommendations for setting the swappiness, or propensity to swap out a process, on a Hadoop system has been to disable swap altogether. With newer versions of the Linux kernel, Out Of Memory (OOM) situations can be more likely to indiscriminately kill important processes to reclaim valuable physical memory on the system with a swappiness of 0. In order to prevent the system from swapping processes too frequently, but still allow for emergency swapping (instead of killing processes), the recommendation is now to 
   
   Set  swappiness to 1 on Linux systems. This will still allow swapping, but with the least possible aggressiveness (for comparison, the default value for swappiness is 60).sysctl command is used to modify kernel parameters at runtime.

* To change the swappiness on a running machine, use the following command:  
  echo 1 > /proc/sys/vm/swappiness       

* To ensure the swappiness is set appropriately on reboot, use the following command:  
    sysctl -w vm.swappiness=1 (-w option is used to make changes to sysctl)     
  
* verify vm.swappiness is set to 1     
  cat /proc/sys/vm/swappiness     
  1  
  
### 2. Disable transparent hugepage compaction (THC)
* To see whether transparent hugepages are enabled, run the following commands and check the output.
  * [always] never means that transparent hugepages is enabled.    
  * always [never] means that transparent hugepages is disabled.   
      cat /sys/kernel/mm/redhat_transparent_hugepage/defrag    
      cat /sys/kernel/mm/redhat_transparent_hugepage/enabled  

* To disable transparent hugepage temporarily as root:     
    echo 'never' > /sys/kernel/mm/redhat_transparent_hugepage/defrag   
    echo 'never' > /sys/kernel/mm/redhat_transparent_hugepage/enabled    
* To disable transparent hugepages temporarily using sudo:    
    sudo sh -c "echo 'never' > defrag_file_pathname"   
    sudo sh -c "echo 'never' > enabled_file_pathname"    

* To disable THC on reboot, add following commands to the /etc/rc.d/rc.local file on all cluster hosts      
* RHEL/CentOS 7.x:    
      echo never > /sys/kernel/mm/transparent_hugepage/enabled    
      echo never > /sys/kernel/mm/transparent_hugepage/defrag    
* RHEL/CentOS 6.x:        
    echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag    
    echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled    

* Verify if transparent hugepages is enabled.   (remove redhat_ for centos 7)      
  cat /sys/kernel/mm/redhat_transparent_hugepage/defrag    
  cat /sys/kernel/mm/redhat_transparent_hugepage/enabled    
    always madvise [never]  

### 3. Make sure that all hosts in the cluster have the same version of OS
    To find Linux Distribution version
        uname - a
          Linux <FQDN> 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
    To find CENTOS or RedHat release 
        cat /etc/redhat-release  
          CentOS Linux release 7.3.1611 (Core)   
        cat /proc/version 
              Linux version 3.10.0-514.26.2.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 
              (Red Hat 4.8.5-11) (GCC) ) #1 SMP Tue Jul 4 15:04:05 UTC 2017 . 
        cat /etc/issue

### 4. Make sure the OS version is supported for the version of CM, CDH, (KMS, KTS if HDFS encryption is required)
    https://www.cloudera.com/documentation/enterprise/release-notes/topics/rn_consolidated_pcm.html#install_cdh_file_system


### 5. Show the mount attributes of your volume(s)
*  Use findmnt command for list of all mounts in the system.   
  Other commands that can be used include 'cat /proc/mounts' and 'etc/mtdfab' 
* To display information about mounted file systems, enter:  
    mount | column -t   
    mount -l (lists all mounts on the machine)  
* df command (difk file usage. Option -h: in human readable form  
    To find out file system disk space usage, enter:     
    $ df      
    $ df -h        
    $ df -ah (include dummy file systems) . 
    $ df -ah --total (display a total line at bottom of listing) .    
    $ df -Th (TYPE of mount of file system eg:ext4, etx3 etc)          
    $ df -Tht ext4 (this will display only ext4) . 
    $ df -Thx ext4 (show all file systems except ext4) . 
* du Command   
    Use the du command to estimate file space usage, enter:    
    du   
    du /home   
    du -ch /home   
* List the Partition Tables      
    The fdisk -l command lists all existing available disk partition on the system (must be run as root):   
    fdisk -l   
    fdisk -l /dev/sda   
* cpu info
    cat /proc/cpuinfo | grep processor
* cpu cores
   cat /proc/cpuinfo | grep core
* Memory 
  cat /proc/meminfo | grep MemTotal
* Partitions
   cat /proc/partitions
* Blocks
   lsblk

OS related commands
1.  uname - a
2. cat /etc/redhat-release  

### 6. If you have ext-based volumes, list the reserve space setting.  
      XFS volumes do not support reserve space.
      The partitions that contains the ext3 and ext4 filesystems reserve the 5% of the total size of the filesystm by 
      default. The idea here is even when you run out of disk space, the root user should still be able to log in and 
      system services should still run. Without this option, the root user could be not able to acces and “clean up” 
      since the system may become unstable, trying to log to in a filesytem full at 100%, for example. The other reason 
      is to help the general optimization with less fragmentation of the filesystem.
      
    df -TH
        Filesystem     Type   Size  Used Avail Use% Mounted on
        /dev/sda1      ext4   106G  1.9G   99G   2% /
        tmpfs          tmpfs  6.8G     0  6.8G   0% /dev/shm

    dumpe2fs -h /dev/sda1 | grep -i block (You can also use tune2fs -l /dev/sda1 | grep 'Reserved block count')
        dumpe2fs 1.41.12 (17-May-2010)  
        Block count:              26213807  
        Reserved block count:     1310368  
        Free blocks:              25357788  
        First block:              0  
        Block size:               4096  
        Reserved GDT blocks:      633  
        Blocks per group:         32768  
        Inode blocks per group:   512  
        Flex block group size:    16  
        Reserved blocks uid:      0 (user root)  
        Reserved blocks gid:      0 (group root)  
        Journal backup:           inode blocks   



### 7. List your network interface configuration    
    ifconfig 
          eth0   Link encap:Ethernet  HWaddr 42:01:AC:1F:75:C7    
                  inet addr:172.31.117.199  Bcast:172.31.117.199  Mask:255.255.255.255  
                  inet6 addr: fe80::4001:acff:fe1f:75c7/64 Scope:Link  
                  UP BROADCAST RUNNING MULTICAST  MTU:1460  Metric:1  
                  RX packets:6760 errors:0 dropped:0 overruns:0 frame:0  
                  TX packets:6924 errors:0 dropped:0 overruns:0 carrier:0  
                  collisions:0 txqueuelen:1000   
                  RX bytes:35643003 (33.9 MiB)  TX bytes:700431 (684.0 KiB)  
          lo     Link encap:Local Loopback    
                  inet addr:127.0.0.1  Mask:255.0.0.0  
                  inet6 addr: ::1/128 Scope:Host  
                  UP LOOPBACK RUNNING  MTU:65536  Metric:1  
                  RX packets:0 errors:0 dropped:0 overruns:0 frame:0  
                  TX packets:0 errors:0 dropped:0 overruns:0 carrier:0  
                  collisions:0 txqueuelen:0   
                  RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)  

      ip link show   
         1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00  
         2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP qlen 1000 
          link/ether 42:01:ac:1f:75:c7 brd ff:ff:ff:ff:ff:ff    

### 8. Show that forward and reverse host lookups are correctly resolved. 
     
    
    If nslookup does not exist it install using
            yum install bind-utils  

    host `hostname`
            bp101.cloudera.com has address 10.20.195.121
    host 10.20.195.121
            121.195.20.10.in-addr.arpa domain name pointer bp101.cloudera.com
            
    nslookup ssubramaniam-bc-fce-1 
       Name:	ssubramaniam-bc-fce-1.gce.cloudera.com 
       Address: 172.31.117.199 
       nslookup 172.31.117.199 
       199.117.31.172.in-addr.arpa	name = ssubramaniam-bc-fce-1.gce.cloudera.com.  

    dig <FQDN>
### 8b. Hostnames should be set to the fully-qualified domain name (FQDN). 
    It is important to note that using Kerberos requires the use of FQDNs, which is important for enabling security 
    features such as TLS encryption and Kerberos. You can verify his with
    
       hostname --fqdn
       
       if using /etc/hosts, ensure that you are listing them in the appropriate order.
               192.168.2.1 bp101.cloudera.com bp101 master1
               192.168.2.2 bl102.cloudera.com bp102 master2


### 9. Check if nscd service is running. 
    ncsd is a nameservice that is used for caching service name lookups
    If needed, run the following commands to start service. Config files are located in /etc/nscd.conf
       systemctl enable nscd.service
       systemctl restart nscd.service

    ps -ef | grep nscd     
        nscd: already running    

    Check if nscd will run on restart   
         chkconfig | grep nscd    
            nscd           	0:off	1:off	2:on	3:on	4:on	5:on	6:off

### 10 .ntpstat - show network time synchronization status.
    ntpstat   
        synchronised to NTP server (169.254.169.254) at stratum 3    
        time correct to within 62 ms  
        polling server every 512 s  


    ntpq – standard NTP query program. \The ntpq utility program is used to monitor NTP daemon ntpd operations and determine performance. The program can be run either in interactive mode or controlled using command line arguments. Type the following command.The ntpq utility program is used to query NTP servers which implement the recommended NTP mode 6 control message format about current state and to request changes in that state. The program may be run either in interactive mode or controlled using command line arguments.   
   * ntpq -pn    
   * remote           refid      st t when poll reach   delay   offset  jitter  
   * +172.17.2.3      71.19.144.130    3 u  456  512  377   47.834   42.682   0.218  
   * +172.17.2.4      45.79.111.114    3 u  277  512  377   47.923   41.738   0.085   
   * 
   * Approximate clock synchronization between the machines on the network can be set up using a service such as ntpd.

    Steps to Setup and install NTP if ntp is not installed
    ------------------------------------------------------
    yum -y install ntp
    ntpdate 0.rhel.pool.ntp.org
    systemctl start  ntpd.service
    systemctl enable ntpd.service

### 11. Disable SELinux.    
    Disabling requires a reboot but using setenforce 0 will make it permissive and not block
      sestatus    
          SELinux status:            enabled
      cd /etc/selinux      
      vi config    
         SELINUX=disabled       
      sestatus   
      setenforce 0   
      sestatus

### 12. Shut off iptables 
https://www.unixmen.com/iptables-vs-firewalld/
     iptables --list (or iptables -L)   
     iptables -F (will flush all tables)   
     service iptables status (only if iptables is running as a service whih it may not sometimes)  
     service iptables stop  
     iptables --list  
     chkconfig iptables off  
     
### 14. Disk Directory permissions

    This is a minor point but you should consider changing the permissions on your directories to 700 before you mount data
    drives. Consequently, if the drives become unmounted, the processes writing to these directories will not fill up the OS
    mount.

   



