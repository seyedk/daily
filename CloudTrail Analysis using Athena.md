## Comming soon!

DDL to create: 
```
CREATE EXTERNAL TABLE cloudtrail_logs (
eventversion STRING,
userIdentity STRUCT<
  type:STRING,
  principalid:STRING,
  arn:STRING,
  accountid:STRING,
  invokedby:STRING,
  accesskeyid:STRING,
  userName:STRING,
  sessioncontext:STRUCT<
    attributes:STRUCT<
      mfaauthenticated:STRING,
      creationdate:STRING>,
    sessionIssuer:STRUCT<
      type:STRING,
      principalId:STRING,
      arn:STRING,
      accountId:STRING,
      userName:STRING>>>,
eventTime STRING,
eventSource STRING,
eventName STRING,
awsRegion STRING,
sourceIpAddress STRING,
userAgent STRING,
errorCode STRING,
errorMessage STRING,
requestParameters STRING,
responseElements STRING,
additionalEventData STRING,
requestId STRING,
eventId STRING,
resources ARRAY<STRUCT<
  ARN:STRING,accountId:
  STRING,type:STRING>>,
eventType STRING,
apiVersion STRING,
readOnly STRING,
recipientAccountId STRING,
serviceEventDetails STRING,
sharedEventID STRING,
vpcEndpointId STRING
)
ROW FORMAT SERDE 'com.amazon.emr.hive.serde.CloudTrailSerde'
STORED AS INPUTFORMAT 'com.amazon.emr.cloudtrail.CloudTrailInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://<Your CloudTrail s3 bucket>/AWSLogs/<optional – AWS_Account_ID>/';
```
a sample query

```
select useridentity.username , sourceipaddress, eventtime,eventname, eventsource,  additionaleventdata
from sampledb.cloudtrail_logs
where eventname <> 'ConsoleLogin'

and useridentity.username = 'ZLK0210220'

and eventtime >= '2017-09-08T00:00:00Z'
and eventtime < '2017-09-10T00:00:00Z';
```