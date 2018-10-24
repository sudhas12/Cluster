### PROVISION A AWS CLUSTER AND INSTALL CLOUDERA DIRECTOR
Reference: https://www.cloudera.com/documentation/director/latest/topics/director_get_started.html

##### Log into AWS using credentials

##### Setting up the AWS Environment
#####  Setting Up a VPC - Setting  up NEW VPC
 * Log in to the AWS Management Console and make sure you are in the desired region. The current region is displayed in the upper-right corner of the AWS Management Console. Click the region name to change your region.
 * In the AWS Management Console, select VPC in the Networking section.
 * Click Start VPC Wizard. (Click VPC Dashboard in the left side pane if the Start VPC Wizard button is not displayed.)
 * Select the desired VPC configuration. For the easiest way to get started, select VPC with a Single Public Subnet.
 * Make sure that DNS Hostnames is set to Yes in the Edit DNS Hostnames dialog.
 *Complete the VPC wizard and then click Create VPC

#####  You can also use an existing VPC. Make sure DNS hostnames is set to 'Y' in the Edit DNS Hostnames Dialog

##### Create a new Security Group
* In the left pane, click Security Groups.
* Click Create Security Group.
* Enter a name and description. Make sure to select the VPC you created from the VPC list box.
* Click Yes, Create.
* Select the newly created security group and add inbound rules as detailed in the table above.

The configured security group should look similar to the following, but with your own values in the Source column.       

   Type        Protocol        PortRange              Source     
    ALL        Traffic ALL	   ALL	                  security_group_id     
    SSH (22)	 TCP (6)	       22	                    <your ip address>      

##### Creating an SSH Key Pair
*To interact with the cluster launcher and other instances, you must create an SSH key pair or use an existing EC2 key pair.
* Select EC2 in Compute section of the AWS console.
* In the Network & Security section of the left pane, click Key Pairs.
* Click Create Key Pair. In the Create Key Pair dialog box, enter a name for the key pair and click Create.
* Note down the key pair name.
* Move the automatically downloaded private key file (with the .pem extension) to a secure location and note the location.
* Change the permission on the pem file to 400

#####Launching an EC2 Instance for Cloudera Director
* To create the instance, in the AWS Management Console, select EC2 from the Services navigation list box in the desired region.
* Click the Launch Instance button in the Create Instance section of the EC2 dashboard.
* Select Community AMIs in the left pane and the latest release of the desired supported distribution. (centos-7.3 ami-46* is one option.
* Select the instance type for Cloudera Director. Cloudera recommends using c3.large or c4.large instances.
* Click Next: Configure Instance Details.
* Select the correct VPC and subnet.
* The cluster launcher requires Internet access; from the Auto-assign Public IP list box, select Enable.
* Use the default shutdown behavior, Stop.
* Click the Protect against accidental termination checkbox.
* (Optional) Click the IAM role drop-down list and select an IAM role.
* Click Next: Add Storage. Cloudera Director requires a minimum of 8 GB.
* Click Next: Add Tags. On the Add Tags page, click the Add Tag button. Enter key owner <uid>. Enter tag stack <development>
* Click Next: Configure Security Group and add ports to existing security group previously created.
* The following ports must be open for the Cloudera Director EC2 instance:      

         Type	        Protocol	     PortRange	  Source     
         SSH (22)	    TCP (6)	        22	        <your ip address>     
         ALL Traffic	ALL	            ALL	         security_group_id   

 * Click Review and Launch. Scroll down to review the AMI details, instance type, and security group information, and then click Launch.
   At the prompt for a key pair:
      Select Choose an existing key pair and select the key pair you created in Setting up the AWS Environment.
      Click the check box to acknowledge that you have access to the private key.

#####RHEL 7 and CentOS 7
* SSH as ec2-user into the EC2 instance you created for Cloudera Director. If you have VPN or AWS Direct Connect, SSH to your private IP address.
   Otherwise, use your public IP address.
*   ssh -i your_file.pem ec2-user@private_IP_address (user will be centos for example for CENTOS image)
*   sudo yum install wget
*   Copy the download link of jdk-8u102-linux-x64.rpm and wget it.
     wget --header "Cookie: oraclelicense=accept-securebackup-cookie" 
    http://download.oracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jdk-8u144-linux-x64.rpm
*   Install with yum localinstall:      
       sudo yum localinstall jdk-8u144-linux-x64.rpm      
* Download the Cloudera director repo file:     
    wget http://archive.cloudera.com/director/redhat/7/x86_64/director/cloudera-director.repo -O /etc/yum.repos.d/cloudera-director.repo
* Install Cloudera Director server and client by running the following command:     
       yum install cloudera-director-server cloudera-director-client     
*  Start the Cloudera Director server by running the following command:    
      sudo service cloudera-director-server start    
* If the RHEL 7 or CentOS firewall is running on the EC2 instance where you have installed Cloudera Director, disable and stop the firewall with the   following commands:     
     sudo systemctl disable firewalld    
     sudo systemctl stop firewalld   

##### Configuring a SOCKS Proxy for Amazon EC2
* In AWS, the security group that you create and specify for your EC2 instances functions as a firewall to prevent unwanted access to your cluster and Cloudera Manager. For security, Cloudera recommends that you not configure security groups to allow internet access to your instances on the instances' public IP addresses. Instead, connect to your cluster and to Cloudera Manager using a SOCKS proxy server.    
* A SOCKS proxy server allows a client (such as your web browser) to connect directly and securely to a server (such as your Cloudera Director server web UI) and, from there, to the web UIs on other IP addresses and ports in the same subnet, including the Cloudera Manager and Hue web UIs. The SOCKS proxy provides you with access to the Cloudera Director UI, Cloudera Manager UI, Hue UI, and any other cluster web UIs without exposing their ports outside the subnet.    
* Note: The same result could be achieved by configuring an SSH tunnel from your browser to the EC2 instance. But an SSH tunnel enables traffic from a single client (IP address and port) to a single server (IP address and port), so this approach would require you to configure a separate SSH tunnel for each connection.   

* To set up a SOCKS proxy for your web browser, follow the steps below.
  * Set Up a SOCKS Proxy Server with SSH
  Set up a SOCKS proxy server with SSH to access the EC2 instance running Cloudera Director. For example, run the following command (with your instance information):

      nohup ssh -i "your-key-file.pem" -CND 8157 ec2-user@instance_running_director_server &
      where

      nohup (optional) is a POSIX command to ignore the HUP (hangup) signal so that the proxy process is not terminated automatically if the terminal process is later terminated.
      your-key-file.pem is the private key you used to create the EC2 instance where Cloudera Director is running.
      C sets up compression.
      N suppresses any command execution once established.
      D 8157 sets up the SOCKS 5 proxy on the port. (The port number 8157 in this example is arbitrary, but must match the port number you specify in your browser configuration in the next step.)

      ec2-user is the AMI username for the EC2 instance where Cloudera Director is running. The AMI username can be found in the details for the instance displayed in the AWS Management Console on the Instances page under the Usage Instructions tab.    
      instance_running_director_server is the private IP address of the EC2 instance running Cloudera Director server, if your networking configuration provides access to it, or its public IP address if not.
       & (optional) causes the SSH connection to run as an operating system background process, independent of the command shell. (Without the &, you leave your terminal open while the proxy server is running and use another terminal window to issue other commands.)
       Important: If you are using a PAC file, the port specified in the PAC file must match the port used in the ssh command (port 8157 in the example above).

##### Configure Your Browser to Use the Proxy   
* Setting Up SwitchyOmega on the Google Chrome Browser     

      Open Google Chrome and go to Chrome Extensions.   
      Search for Proxy SwitchyOmega and add to it Chrome.   
      In the Profiles menu of the SwitchyOmega Options screen, click New profile and do the following:   
      In the Profile Name field, enter AWS-Cloudera.   
      Select the type PAC Profile.   
      The proxy autoconfig (PAC) script contains the rules required for Cloudera Director. Enter or copy the following into the PAC Script field:    


          function regExpMatch(url, pattern) {    
            try { return new RegExp(pattern).test(url); } catch(ex) { return false; }    
          }

          function FindProxyForURL(url, host) {
              // Important: replace 172.31 below with the proper prefix for your VPC subnet

              if (shExpMatch(url, "*172.31.*")) return "SOCKS5 localhost:8157";
              if (shExpMatch(url, "*ec2*.amazonaws.com*")) return 'SOCKS5 localhost:8157';
              if (shExpMatch(url, "*.compute.internal*") || shExpMatch(url, "*://compute.internal*")) return 'SOCKS5 localhost:8157';
              if (shExpMatch(url, "*ec2.internal*")) return 'SOCKS5 localhost:8157';
              return 'DIRECT';
          }
In the Actions menu, click Apply Changes.  
* On the Chrome toolbar, select the AWS-Cloudera profile for SwitchyOmega.
* Open a web browser and go to the private IP address of the instance you created in Launching an EC2 Instance for Cloudera Director. Include port 7189 in the address. For example:
http://192.0.2.0:7189
