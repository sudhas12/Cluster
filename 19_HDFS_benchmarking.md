
### Hadoop Benchmarks
-------------------------------
Hadoop benchmarks provide insight into cluster performance. terasort and TestDFSIO, provide a good sense of how well a Hadoop installation is operating and can be compared with public data published for other Hadoop systems. The results, should not be taken as a single indicator for system-wide performance on all applications. The best benchmarks are always 

Reference: 

           http://www.informit.com/articles/article.aspx?p=2453563&seqNum=2
           https://gist.github.com/ace-subido/0a9b219b2348921f6a87
           http://www.michael-noll.com/blog/2011/04/09/benchmarking-and-stress-testing-an-hadoop-cluster-with-terasort-testdfsio-nnbench-mrbench/ 
           http://www.maninmanoj.com/2015/05/bench-marking-tools-in-hadoop.html

#### Create a folder where the benchmark result files are saved
      sudo -u hdfs mkdir /home/hdfs/benchmark

#### Run as hdfs user (Give permissions if otherusers would like to run benchmark tests)
      sudo -u hdfs chmod 777 /home/hdfs/benchmark
  
#### TestDFSIO writes results to local directory.(Give permissions to hdfs to write to a local home directory
      chmod 777 /home/$user (is one option)

### Teragen, Teraset and Teravalidate benchmark
-------------------------------
The teragen, terasort and teravalidate are designed for full Hadoop cluster installations and assume a multi-disk HDFS environment. The terasort benchmark sorts a specified amount of randomly generated data. This benchmark provides combined testing of the HDFS and MapReduce layers of a Hadoop cluster. It consists of following 3 steps:
         
         Generating the input data via teragen program.
         Running the actual terasort benchmark on the input data.
         Validating the sorted output data via the teravalidate program.

In general, each row is 100 bytes long; thus the total amount of data written is 100 times the number of rows specified as part of the benchmark (i.e., to write 100GB of data, use 1 billion rows). The input and output directories need to be specified in HDFS. The following sequence of commands will run the benchmark for 50GB of data as user hdfs. Make sure the /user/hdfs directory exists in HDFS before running the benchmarks.

#### Run teragen to generate rows of random data to sort. 
This creates a 500GB, 5GB and 10GB files HDFS under the /benchmarks/terasort-input folder. It's for the terasort to run it's benchmark into.

      time hadoop jar hadoop-mapreduce-examples.jar teragen 5000000000 /benchmarks/terasort-500GB-input // 500GB
      time hadoop jar hadoop-mapreduce-examples.jar teragen 50000000 /benchmarks/terasort-5GB-input // 5GB
      time hadoop jar hadoop-mapreduce-examples.jar teragen 100000000 /benchmarks/terasort-10GB-input // 10GB
#### Run terasort to sort the data.      
This command runs a benchmarking mapreduce job on the data created by teragen in the /benchmarks/terasort-input folder. The output result of the reduce is written into files and placed under the /benchmarks/terasort-output/ folder, where teravalidate will check later. 
           
     time hadoop jar hadoop-mapreduce-examples.jar terasort /benchmarks/terasort-input /benchmarks/terasort-output

#### terasort to use 4 reducer tasks .     

   time  hadoop jar hadoop-mapreduce-examples.jar terasort -D mapred.reduce.tasks=4 /benchmarks/terasort-input /benchmarks/terasort-output-tasks4
    
#### Run teravalidate to validate the sort.    
This command just ensures if the output of terasort was valid and without error.
     hadoop jar hadoop-mapreduce-examples.jar teravalidate /benchmarks/terasort-output /benchmarks/terasort-validate

To report results, the time for the actual sort (terasort) is measured and the benchmark rate in megabytes/second (MB/s) is calculated. For best performance, the actual terasort benchmark should be run with a replication factor of 1. In addition, the default number of terasort reducer tasks is set to 1. Increasing the number of reducers often helps with benchmark performance.


    

 
#### Cleanup 
           hdfs dfs -rm -r -skipTrash Tera*

### TestDFSIO Benchmark
-------------------------------
The TestDFSIO tool is used to test the I/O performance of HDFS.It is useful in finding the bottlenecks in network, hardware, 
The TestDFSIO benchmark is a read and write test for HDFS. That is, it will write or read a number of files to and from HDFS and is designed in such a way that it will use one map task per file. The file size and number of files are specified as command-line arguments. Similar to the terasort benchmark, you should run this test as user hdfs.

Similar to terasort, TestDFSIO has several steps. In the following example, 16 files of size 1GB are specified. Note that the TestDFSIO benchmark is part of the hadoop-mapreduce-client-jobclient.jar. Other benchmarks are also available as part of this jar file. Running it with no arguments will yield a list. In addition to TestDFSIO, NNBench (load testing the NameNode) and MRBench (load testing the MapReduce framework) are commonly used Hadoop benchmarks. Nevertheless, TestDFSIO is perhaps the most widely reported of these benchmarks. The steps to run TestDFSIO are as follows:

Run TestDFSIO in write mode to create data.

Usage: hadoop jar $HADOOP_HOME/hadoop-*test*.jar TestDFSIO 

           -read | -write | -clean 
           [-nrFiles N] [-fileSize MB]
           [-resFile resultFileName] [-bufferSize Bytes]

           The buffer size flag describes the length of the write buffer in bytes. See the WriteMapper 
           nrFiles ==> number of files to create
           fileSize  (which can be i KB,MB,GB,TB (Default is MB) is size of each file created

### Run -write
This automatically creates a folder under HDFS folder /benchmarks/TestDFSIO where it writes out these files

            hadoop jar $HADOOP_EXAMPLES/hadoop-mapreduce-client-jobclient-tests.jar TestDFSIO -write  -nrFiles 16 -fileSize 1000

Example results are as follows (date and time prefix removed).

           fs.TestDFSIO: ----- TestDFSIO ----- : write
           fs.TestDFSIO:            Date & time: Thu May 14 10:39:33 EDT 2015
           fs.TestDFSIO:        Number of files: 16
           fs.TestDFSIO: Total MBytes processed: 16000.0
           fs.TestDFSIO:      Throughput mb/sec: 14.890106361891005
           fs.TestDFSIO: Average IO rate mb/sec: 15.690713882446289
           fs.TestDFSIO:  IO rate std deviation: 4.0227035201665595
           fs.TestDFSIO:     Test exec time sec: 105.631

### Run -read
Run TestDFSIO in read mode. This reads the files produced by the -write step in HDFS folder to read throughput

           hadoop jar  $HADOOP_EXAMPLES/hadoop-mapreduce-client-jobclient-tests.jar TestDFSIO -read  -nrFiles 16 -fileSize 1000
Example results are as follows (date and time prefix removed). The large standard deviation is due to the placement of tasks in the cluster on a small four-node cluster.

           fs.TestDFSIO: ----- TestDFSIO ----- : read
           fs.TestDFSIO:            Date & time: Thu May 14 10:44:09 EDT 2015
           fs.TestDFSIO:        Number of files: 16
           fs.TestDFSIO: Total MBytes processed: 16000.0
           fs.TestDFSIO:      Throughput mb/sec: 32.38643494172466
           fs.TestDFSIO: Average IO rate mb/sec: 58.72880554199219
           fs.TestDFSIO:  IO rate std deviation: 64.60017624360337
           fs.TestDFSIO:     Test exec time sec: 62.798

### Clean up the TestDFSIO data.
Cleans out the /benchmarks/TestDFSIO folder in HDFS

           hadoop jar  $HADOOP_EXAMPLES/hadoop-mapreduce-client-jobclient-tests.jar TestDFSIO -clean

Ref: http://www.michael-noll.com/blog/2011/04/09/benchmarking-and-stress-testing-an-hadoop-cluster-with-terasort-testdfsio-nnbench-mrbench/
Here, the most notable metrics are Throughput mb/sec and Average IO rate mb/sec.
Both of them are based on the file size written (or read) by the individual map tasks and the elapsed time to do so (see this discussion thread for more information).

Throughput mb/sec for a TestDFSIO job using N map tasks is defined as follows. 
The index 1 <= i <= N denotes the individual map tasks:

Throughput(N)=∑Ni=0filesizei∑Ni=0timei
Throughput(N)=∑i=0Nfilesizei∑i=0Ntimei
Average IO rate mb/sec is defined as:

Average IO rate(N)=∑Ni=0rateiN=∑Ni=0filesizeitimeiN
Average IO rate(N)=∑i=0NrateiN=∑i=0NfilesizeitimeiN
Two derived metrics you might be interested in are estimates of the “concurrent” throughput and average IO rate (for the lack of a better term) your cluster is capable of. Imagine you let TestDFSIO create 1,000 files but your cluster has only 200 map slots. This means that it takes about five MapReduce waves (5 * 200 = 1,000) to write the full test data because the cluster can only run 200 map tasks at the same time. In this case, simply take the minimum of the number of files (here: 1,000) and the number of available map slots in your cluster (here: 200), and multiply the throughput and average IO rate by this minimum. In our example, the concurrent throughput would be estimated at 4.989 * 200 = 997.8 MB/s and the concurrent average IO rate at 5.185 * 200 = 1,037.0 MB/s.

Another thing to keep in mind when interpreting TestDFSIO results is that the HDFS replication factor plays an important role. If you compare two otherwise identical TestDFSIO write runs which use an HDFS replication factor of 2 and 3, respectively, you will see higher throughput and higher average IO numbers for the run with the lower replication factor. Unfortunately, you cannot simply overwrite your cluster’s default replication value on the command line via

### Understating the  result:

           http://www.maninmanoj.com/2015/05/bench-marking-tools-in-hadoop.html

The important parameters in TestDFSIO are the important parameters to note are "Throughput" and "Average IO rate".
NOTE: Replication factor of the cluster plays an important role in managing these values. For example, a cluster having 
an replication factor of "2" will have better throughput and I/O as compared to a cluster with replication factor of "3".
Suppose, you want to test this by changing the default replication factor of "3". 

TeraGen/TeraSort/TeraValidate:

It is a benchmark that combines testing the HDFS and MapReduce layers of an Hadoop cluster. 

We can use the TeraSort benchmark on our Hadoop configuration if the cluster passed a TestDFSIO benchmark. TeraSort is helpful to determine whether map and reduce slot assignments are sound (as they depend on the variables such as the number of cores per TaskTracker node and the available RAM), whether other MapReduce-related parameters such as io.sort.mb and mapred.child.java.opts are set to proper values.

TeraGen:  It generates a random output data.  The syntax is as follows:
------
hadoop jar hadoop-*examples*.jar teragen <number of 100-byte rows> <output dir>
-----

Example: 
--------
hadoop jar hadoop-*examples*.jar teragen 5242880 /data/input
--------

The above command will produce "500 MB' file to directory "/data/input".

The actual TeraGen data format per row is:

-------------
<10 bytes key><10 bytes rowid><78 bytes filler>\r\n
-------------

1) The keys are random characters from the set ' '..'~'.
2) The rowid is the right justified row id as a int.
3) The filler consists of 7 runs of 10 characters from ‘A’ to ‘Z’.

Suppose, the Blocksize is set to 128MB, then teragen will get complete in a short span of time.
Sometimes, you may have to do testing my increasing the blocksize to 512MB, then it can be done via command line itself:

-----------
hadoop jar hadoop-*examples*.jar teragen -D dfs.block.size=536870912  5242880 /data/input
------------

The above command will produce 500 MB data, with 512 MB block size.


TeraSort: 

TeraSort: Run the actual TeraSort benchmark

The syntax is as follows:
----------
hadoop jar hadoop-*examples*.jar terasort <input dir> <output dir>
----------

For example:
-------
hadoop jar hadoop-*examples*.jar terasort /data/input /data/output
------

In the above example, the input dir is "/data/input" and output directory is "/data/output" in HDFS.


TeraValidate: Validate the output data of terasort.

The syntax for TeraValidate:
-----
hadoop jar hadoop-*examples*.jar teravalidate <terasort output dir (= input data)> <teravalidate output dir>
-----

Example:
--------
hadoop jar hadoop-*examples*.jar teravalidate /data/output /data/teravalidate-output
--------

If everything is properly sorted then, teravalidate will not produce any output in "/data/teravalidate-output".

From the command line, we can get the details of the completed job as below:

Syntax:
--------
hadoop job -history all <job output directory>
--------

Example:
--------
hadoop job -history all /data/output/
--------

Parameter 3: mrbench ( MapReduce Benchmark).

MRBench the source code is src/test/org/apache/hadoop/mapred/MRBench.java. It loops a small job a number of times. It is a very complimentary benchmark to the "large-scale" TeraSort benchmark because MRBench checks whether small job runs are responsive and running efficiently on a cluster. Its focus is on the MapReduce layer as its impact on the HDFS layer is very limited.


The available options of  "mrbench" can be interpreted as below:



The command to run a loop of small 3 test job is:
---------
hadoop jar hadoop-*test*.jar mrbench -numRuns 3
---------

The output of the above command is as below:



This means that, the average time to complete the job is 21 seconds.


Parameter 4: nnbench for NameNode benchmark.

This is useful for load testing of  NameNode hardware and configuration. It generates a lot of HDFS-related requests for the sole purpose of putting a high stress on the NameNode. The benchmark can simulate requests for creating, reading, renaming and deleting files on HDFS.

The available options for the tool are, 



Command:
--------
hadoop jar hadoop-*test*.jar nnbench -operation create_write -maps 12 -reduces 6 -blockSize 1 -bytesToWrite 0 -numberOfFiles 1000 -replicationFactorPerFile 3 -readFileAfterOpen true -baseDir /benchmarks/NNBench-`hostname -s`
---------

The above command will run a NameNode benchmark that creates 1000 files using 12 maps and 6 reducers. It uses a custom output directory based on the machine’s short hostname. This is a simple trick to ensure that one box does not accidentally write into the same output directory of another box running NNBench at the same time.

Reference links:
-------
http://www.michael-noll.com/blog/2011/04/09/benchmarking-and-stress-testing-an-hadoop-cluster-with-terasort-testdfsio-nnbench-mrbench/#teravalidate-validate-the-sorted-output-data-of-terasort

http://epaulson.github.io/HadoopInternals/benchmarks.html#nnbench
-------


