#### JAVA JDK 8 Installation STEPS for CDH Install 
  https://www.cloudera.com/documentation/enterprise/5-10-x/topics/cdh_ig_jdk_installation.html
  Install the same version of the Oracle JDK on each host.    
  Install the JDK in /usr/java/jdk-version.    
  
    Copy the download link of jdk-8u102-linux-x64.rpm and wget it.    
    Go to http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html and       
    copy the correct. link below may not work        
          wget --header "Cookie: oraclelicense=accept-securebackup-cookie"     
          http://download.oracle.com/otn-pub/java/jdk/8u102-b14/jdk-8u102-linux-x64.rpm  . 

    Installation Method 1:     
    ----------------------     
          * Install the rpm   
             rpm -ivh \<jdk8rpm\>.rpm

          *  Verify rpm was installed     
             rpm -qa | grep jdk    
   
    Installation  Method 2:      
    -----------------------
        sudo yum localinstall \<rpm_name>    For example: yum localinstall jre-8u151-linux-x64.rpm        
       
    Installation Method 3:      
    ---------------------     
       Download the .tar.gz file for one of the supported versions of the Oracle JDK from Java SE 8/7 Downloads.    
       Extract the JDK to /usr/java/jdk-version; for example /usr/java/jdk.1.7.0_nn or /usr/java/jdk.1.8.0_nn,     
       where nn is a supported version.   
        
          tar -xzf \<xxx.tar.gz\>   (x => extract ; z=> uncompress f => file (note f option should be last)     
          Copy the extracted jdk directory to /usr/java/\<jdk.xxx\>   
         
     If jdk directory is not copied, then set JAVA_HOME to the directory where the JDK is installed, then     
     add the following line to the file /etc/default/cloudera-scm-server. This affects only the Cloudera Manager    
     Server process, and does not affect the Cloudera  Management Service roles:    
            export JAVA_HOME=/usr/java/jdk.1.8.0_nn    
           
    Custom Java Home location for entire cluster   
    This change affects all CDH processes and Cloudera Management Service roles in the cluster             
            Open the Cloudera Manager Admin Console.   
            In the main navigation bar, click the Hosts tab  
            Click the Configuration tab.    
            Select Category > Advanced.    
            Set the Java Home Directory property to the custom location.    
            Click Save Changes.    
