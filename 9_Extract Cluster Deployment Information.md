#### sec-score project location: http://github.mtv.cloudera.com/tristan/sec-score

##### 1. CURL command to extract deployment information from cluster that has been SECURED
    (use /api/v#/cm/deployment)
    curl -u admin:admin http://localhost:7180/api/v#/cm/deployment?view=export_redacted > /path/to/confg-export.json

##### 2. Alternatively 
    https://<hostname FQDN >:7183/api/v15/cm/deployment?view=EXPORT_REDACTED
    

##### 3.Copy the output of above command to a json file on linux box using vi
    ...   

##### 4. Install git on linux
    yum install git

##### 5.Download Maven tar.gz file 
    wget <download mirror file.gz>

##### 6. Install maven
    tar xzvf apache-maven-3.5.2-bin.tar.gz

##### 7. Add the bin directory of the created directory apache-maven-3.5.2 to the PATH environment variable
     export PATH =<maven dir>:$PATH
     
##### 8. Confirm with mvn -v in a new shell. 
    mvn -n 

##### 9. The result should look similar to
    Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T08:58:13+01:00)
    Maven home: /opt/apache-maven-3.5.2
    Java version: 1.8.0_45, vendor: Oracle Corporation
    Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
    Default locale: en_US, platform encoding: UTF-8
    OS name: "mac os x", version: "10.8.5", arch: "x86_64", family: "mac"

##### 10. Clone sec-score project
    git clone http://github.mtv.cloudera.com/tristan/sec-score.git

##### 11. cd to sec-score donwloaded directory
    cd  sec-score 

##### 12. Build the project
     mvn install
     
##### 13. Run following command against the deployment JSON file out put from step 1 or 3.
    java -jar target/sec-score-0.3-SNAPSHOT-jar-with-dependencies.jar --deploymentjson deploy.json  --outputhtmlfile  results.html
 
