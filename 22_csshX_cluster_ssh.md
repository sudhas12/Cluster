#csshX command can be used at MAC Level to manage multipel linux machines 
  * Note this is not a replacement to using Ansible, SALT or other scripts used when the number of nodes is  large
  * http://macappstore.org/csshx/
  * Mak sure bre is installed on 

   brew install csshx 


#### Cluster ssh (ccsh) can be used to manage multiple Linux machines at the same time
  https://www.linux.com/learn/managing-multiple-linux-servers-clusterssh
  
  
#### Using csshX
csshX -l root sub-1.gce.cloudera.com sub-2.gce.cloudera.com sub-3.gce.cloudera.com sub-4.gce.cloudera.com sub-5.gce.cloudera.com




#####  Install cluster ssh (this is a different command from csshX - thsi requires Display settings and not recommended)
       yum install clusterssh

##### Configuring clusterSSH .  
  * ClusterSSH can be configured either via its global configuration file —    
  * /etc/clusters, or via a file in the user's home directory called .csshrc.   
  
##### Using ClusterSSH Using is similar to launching SSH by itself except you use clustername 
  * cssh -l <username> <clustername>
  
  
 ##### Alternatively, you can also login to machines that aren't in your .csshrc file, 
       simply by running cssh -l <username> <machinename1> <machinename2> <machinename3>.


 pssh command can be used to ssh pararelling to multiple machines and run the same command across all machines
 https://linux.die.net/man/1/pssh - No TERMINAL 
 
 pssh -i -h hosts.txt -A -l root echo hi
