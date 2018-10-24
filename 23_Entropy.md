##### 1.Check entropy
     cat /proc/sys/kernel/random/entropy_avail

##### 2. Install rng tool
     yum install rng-tools

##### 3.Make sure rngd starts on reboot
     chconfig rngd on

##### 4. Start rngd service   
     service rngd start  

##### 5.Check status of rngd service    
     service rngd status 

##### 6. Check if it will automatically start on reboot   
    chkconfig --list rngd 

##### 7.Check entropy again   
    cat /proc/sys/kernel/random/entropy_avail 
