Amazon ADS - Setting UP Amazon Linux AMI to work with ADS Agent

**Background**
Amazon ADS Tool provides and agent (AWSAgent) which can be deployed to the host machine of the enterprise (Linux and Windows workload) to capture the necessary data required for planning on migration of workloads to the Cloud or any data center. 
AWS Agent tool, once deployed to a host (physical or virtual), extracts informations such as the OS, Processes, Resource Utilization and Performance, Inbound and Outbound traffics, and server name. As the gathered information of an organization can grow big, there needs to be a process to gather, analyze, and process these data in an efficient way. The focus of this document is to setup of an environment based on Amazon Linux AMI to work with Amazon ADS Tool for this purpose (of gathering, processing, and analyzing) in a bath process ...
For more details on AWS Application Discovery tool please refer to 
https://aws.amazon.com/application-discovery/
Configuring the environment
Prerequisites
Security Configuration: Setup Identity and Access Management Policy and Role.  
Sign in to the console (pacden.signin.aws.amazon.com/console)  navigate to IAM. create the following policy. for name you may choose **ads-s3-bucket-fullaccess**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1504025864000",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::pds-awsagent-export-2017-08-22",
                "arn:aws:s3:::pds-awsagent-export-2017-08-22/*"
            ]
        }
    ]
}

```
Create an `IAM Role` with the following policies attached. you may call it  `ADS-S3-Access`
1) `AWSAgentlessDiscoveryService`
2) `ads-s3-bucket-fullaccess`
Create a Bucket policy that allows specific users to access the bucket
if you need to share your EC2 private key (.pem file) with the team, it's recommended to keep the key secure. to do so, setup a new S3 Bucket for the key and Attach a policy like the one follows and modify the users properly.
```
{
    "Version": "2012-10-17",
    "Id": "Policy1504110917281",
    "Statement": [
        {
            "Sid": "Stmt1504110910611",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::604993478738:user/<username>",
                    "arn:aws:iam::604993478738:user/<user name>"
                   
                ]
            },
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::bkt-generic-keypairs",
                "arn:aws:s3:::bkt-generic-keypairs/*"
            ]
        }
    ]
}
```
## Setting up the environment:
1.	Launch an Amazon Linux AMI : - The ADS Tool extension is memory intensive process and need 3-4 GB of RAM.  During the lunch ensure the following: 
* The EC2 assumes the role we have created in prerequisites step.
* The EC2 is provisioned in a VPC/Subnet that you can reach to. 
The best practice for instances with sensitive data in a private subnet and using a bastion host to connect to the instance. As the instance we are building does not have any business information and security, it's safe to place it in a public subnet but restrict access using Security Groups and `NACL`s.
2.	Setup Python. To setup python on Linux AMI,  please use [this link](http://docs.aws.amazon.com/cli/latest/userguide/awscli-install-linux-python.html): 
3.	`AWS CLI`, Python (2.7+)
4.	Setup pip by the help of [this link](http://docs.aws.amazon.com/cli/latest/userguide/awscli-install-linux.html): 
NOTE: you need to capture where the installation of the pip happens, specially after upgrading the PIP. The path is typically at `/usr/local/bin`directory. you may add it to the `~/.bashrc` or `~/.bash_profile`. in any of the cases, you may use the following snippet:
`export PATH=$PATH:/usr/local/bin`
and for the current session, you can activate it by `source ~/.bashrc`
5.	Setup `boto` ,`boto3`: 
```
pip install boto
```

```
pip install boto3
```
Install Java 1.8, and ensure it's the default runtime for `java` and `javac`:
On Amazon Linux `AMI`s you should make sure Java 1.8 is enabled: 
```
yum install java-1.8.0
yum install java-1.8.0-openjdk-devel
sudo /usr/sbin/alternatives --config java
sudo /usr/sbin/alternatives --config javac
```
If you prefer, you can remove `java 1.7` with
```
sudo yum remove java-1.7.0-openjdk
```

NOTE: but remove it after you installed Java 1.8 or the `aws-apitools` will also be removed as they depend on Java on being installed

8.	Setup git on the `linux` AMI
9.	Setup `pySpark`
1.	$ pip install `pyspark`
2.	Make sure to configure this:  `$ export SPARK_LOCAL_IP=127.0.0.1`
Using the ADS python code
Process Overview
The process of collecting data starts when a user (or a process) kicks of an agent hosted on a `VM` (or a physical machine) to start collecting data. as soon as the agent starts collecting, it sends the host, process, in & out of network packets, and the other useful information to ADS Discovery Database hosted on `AWS`. This information is accessible from the ADS console for each agent. However, in order to get more elaborate information for all the servers, the ADS console is practically hard to use. For instance, if you want to find out all the dependent host machines to a specific host machine and the type of dependencies, you may not want to use Console, rather you will use the ADS `API` to collect all the host information and ship it to your Big Data solution to run any type of transformation and analytics. 
The purpose of the python code described here is to show how to leverage Python `API`s for `AWS/ADS` to collect the detail information of the ADS Agents and ship them to Amazon's data lake (`s3`) and use an analytic tool (such as Athena) to query the information required. 
In the rest of document, we use the term "export" for when we get the information of ADS Agent(s) from ADS Database and export them to "comma separated value (.csv)" files. We will introduce a simple python code that connects to ADS API and runs ADS Tasks that are purposed for exporting and generating the collected data into CSV and save it locally.
The term "Convert" is used when we conver the collected data into a particular format (in this case `Hadoop` Parquets) and store it to the `data lake`. We have another python code that it's job is simply finding all the CSV files related to a topic (process, host, source and destination, and other useful infos) and upload them to S3.

Let's examine the process. follow these steps:
create a new background Session with Screen command. 
```
screen 
```
Create a new directory where the export command will store the exported files
Run "export". the export python script has the following  syntax:
```
python export.py --directory <directoryName> --start-time <start-time> --end-time <end-time> --filter <"space separated agent-ids">
```
Detach the process by running `CTRL+a+d`
you may come back to visit this session using "$ screen -r"


<directoryName> is the name of the directory you just created 
<start-time> the start time for the range of date /time you want to capture the data collection
<end-time> the end date/time of the range of the data collection.
<filter> (optional) an array (space separated list of agent Ids) to get the collected data for.

The following command exports all of the agents collected data in the range of 3 days (from 14th to 17th of Aug  )  into the `export-2017-08-17` directory under `/home/ec2-user/` directory 
In a typical scenario, you may not need to use a filter and run the command for all of the agents. however, there are times you may need to run the "export" command only for a limited number of agents. the following example shows running the command for 5 agents. 
```
python export.py --directory "<<a directory >>" --start-time 2017-08-19T12:00 --end-time 2017-08-22T12:00 --filter o-cf73a35de95c06c77 o-cf92c64a4965d497b o-cfbd0226fb6597f91 o-cfca7f6775955ed16 o-d089a508260c5142b 
```


After running the export command, you will have all the collected data for the range of days specified in `export.py` command in the directory you specified. In the directory you'll find a new folder called `agentExport` as the root of all the directories for each agent. The next step will be converting them to "parquet" format and shipping it over to the `S3` bucket. 
Here's the syntax using the convert command:
```
python convert_csv.py --directory <directory name> --bucket-name <bucket name>  --region <aws region for the bucket>
```

The following command converts the exported data within the `/f100/` directory to the `s3` bucket named `bkt-awsagent-export-2017-08-22` in the us-west-2 region.


```
python convert_csv.py --directory <<a directory such as /root/f100/>> --bucket-name "bkt-awsagent-export-2017-08-22" --region us-west-2
```
Once the data is shipped to `S3`, you can build use Athena to build the database and tables of the Athena.

## How to setup Athena for ADS Analytics
To perform Analysis on the data gather in the `S3`, you may consider using `AWS Athena`. The previous step tried to load the collected data in to `s3` while transforming it from `CSV` to [Parquets](https://parquet.apache.org/). 

Now we need to create a database in Athena and create the tables with schema generated for the parquets. 

Use the `DDL` in the [link in git](http://github.com/seyedk/aws-discovery-utils/blob/master/discovery_athena.ddl) to create the tables.

So far we have the agent level data. 

There are some data collected that is not tied to the agents, but the overall host, performance, etc.
To get access to this collected data, you should run the `aws discovery` CLI (after you've installed it).

Step 1) run the following command to create an export task, capture the export id
```
aws discovery start-export-task
```

Step 2) Use the export ID you captured in Step 1, to get the S3 Pre-Signed URL that stores the data:

```
aws discovery describe-export-tasks --export-id export-43066a57-ede6-46f8-919c-7dfdd81c6284
```

Copy the s3 singed URL and use a browser to download, extract, rename files and then upload them to the an appropriate s3 location like the one I have used below for the Server `csv` file. 



You need to upload a `csv` to the s3 bucket as highlighted in the DDL below

```
CREATE EXTERNAL TABLE IF NOT EXISTS Server (
  `Id` string,
  `accountNumber` bigint,
  `agentId` string,
  `cpuType` string,
  `hostName` string,
  `hypervisor` string,
  `osName` string,
  `osVersion` string,
  `smBiosId` string,
  `type` string,
  `creationTimeStamp` timestamp,
  `lastModifiedTimeStamp` timestamp,
  `agentProvidedTimeStamp` timestamp 
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
  'serialization.format' = ',',
  'field.delim' = ','
) LOCATION 's3://<bucketName>/Server/'
TBLPROPERTIES ('has_encrypted_data'='false')
```