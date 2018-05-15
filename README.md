# notes


Amazon Kinesis Data Firehose is a fully managed service for delivering real-time streaming data to 
 
    - Amazon Simple Storage Service (Amazon S3)
    - Amazon Redshift
    - Amazon Elasticsearch Service (Amazon ES)
    - Splunk
  

  - With Kinesis Data Firehose, you don't need to write applications or manage resources. You configure your data producers to send data to Kinesis Data Firehose, and it automatically delivers the data to the destination that you specified.

 - You can also configure Kinesis Data Firehose to transform your data before delivering it.

 - For Amazon Redshift destinations, streaming data is delivered to your S3 bucket first. Kinesis Data Firehose then issues an Amazon Redshift COPY command to load data from your S3 bucket to your Amazon Redshift cluster. If data transformation is enabled, you can optionally back up source data to another Amazon S3 bucket.

  You can use the AWS Management Console or an AWS SDK to create a Kinesis data delivery stream to your chosen destination

  You can update the configuration of your Kinesis data delivery stream at any time after it’s created, using the Kinesis Data Firehose    console or UpdateDestination
  
  Your Kinesis data delivery stream remains in the ACTIVE state while your configuration is updated, and you can continue to send data.   The updated configuration normally takes effect within a few minutes. The version number of a Kinesis data delivery stream is          increased by a value of 1 after you update the configuration, and it is reflected in the delivered Amazon S3 object name.
 
  Kinesis Data Firehose buffers incoming data before delivering it to Amazon S3. You can choose a buffer size (1–128  MBs) or buffer interval (60–900 seconds); whichever condition is satisfied first triggers data delivery to Amazon S3. 

  In circumstances where data delivery to the destination falls behind data writing to the delivery stream, Kinesis Data Firehose raises the buffer size dynamically to catch up and ensure that all data is delivered to the destination

  Choose GZIP, Snappy, or Zip data compression, or no data compression. Snappy or Zip compression are not available for delivery streams   with Amazon Redshift as the destination

  Kinesis Data Firehose supports Amazon S3 server-side encryption with AWS Key Management Service (AWS KMS) for encrypting delivered       data in Amazon S3.

  You can choose to not encrypt the data or to encrypt with a key from the list of AWS KMS keys that you own

  When copying data from S3 to Redshift cluster using Redshift COPY command, 
         -  "GZIP" is required if S3 data compression is enabled
         -   "REGION" is required if your S3 bucket isn't in the same AWS Region as your Amazon Redshift cluster
         
  Kinesis Data Firehouse retries for (0-7200) seconds, i.e. max 2 hours if data COPY to Redshift cluster fails

If you enable data transformation with Lambda, you can enable source record backup to deliver untransformed incoming data to a separate S3 bucket. You cannot disable source record backup after you enable it

Kinesis Data Firehose doesn't delete the data from your S3 bucket after loading it to your Amazon Redshift cluster. You can manage the data in your S3 bucket using a lifecycle configuration

Time duration (0–7200 seconds) for Kinesis Firehose to retry if an index request to your Amazon ES cluster fails

After sending data, Kinesis Firehose first waits for an acknowledgment from Splunk. If an error occurs or the acknowledgment doesn’t arrive within the acknowledgment timeout period, Kinesis Firehose starts the retry duration counter. It keeps retrying until the retry duration expires, after which Kinesis Firehose considers it a data delivery failure and backs up the data to your Amazon S3 bucket.

Every time Kinesis Firehose sends data to Splunk, whether it's the initial attempt or a retry, it restarts the acknowledgement timeout counter and waits for an acknowledgement to arrive from Splunk

Even if the retry duration expires, Kinesis Firehose still waits for the acknowledgment until it receives it or the acknowledgement timeout period is reached. If the acknowledgment times out, Kinesis Firehose determines whether there's time left in the retry counter. If there is time left, it retries again and repeats the logic until it receives an acknowledgment or determines that the retry time has expired.




Data sources for Kinesis Data Firehose delivery stream
  
    - Kinesis stream
    - Kinesis Agent
    - Kinesis Firehose API using AWS SDK
    - Amazon CloudWatch  (same region as kinesis data firehose delivery stream)
    - CloudWatch Events  (same region as kinesis data firehose delivery stream)
    - AWS IoT            (same region as kinesis data firehose delivery stream)  
    
    
 Firehose + Kinesis Streams
    
- When you configure a Kinesis stream as the source of a Kinesis Data Firehose delivery stream, the Kinesis Data Firehose PutRecord and PutRecordBatch operations are disabled. 

- To add data to your Kinesis Data Firehose delivery stream in this case, use the Kinesis Data Streams PutRecord and PutRecords operations

- Kinesis Data Firehose starts reading data from the LATEST position of your Kinesis stream

- Kinesis Data Firehose calls the Kinesis Data Streams GetRecords operation once per second for each shard

- More than one Kinesis Data Firehose delivery stream can read from the same Kinesis stream. Other Kinesis applications (consumers) can also read from the same stream. Each call from any Kinesis Data Firehose delivery stream or other consumer application counts against the overall throttling limit for the shard. 



  Firehose + Kinesis Agent

 - Kinesis Agent is a stand-alone Java software application that offers an easy way to collect and send data to Kinesis Data Firehose

 - The agent continuously monitors a set of files and sends new data to your Kinesis data delivery stream

 - The agent handles file rotation, checkpointing, and retry upon failures

 - It also emits Amazon CloudWatch metrics to help you better monitor and troubleshoot the streaming process

 - By default, records are parsed from each file based on the newline ('\n') character. However, the agent can also be configured to parse multi-line records 

 - You can install the agent on Linux-based server environments such as web servers, log servers, and database servers

 - To use Kinesis Agent, your OS must be either Amazon Linux AMI with version 2015.09 or later, or Red Hat Enterprise Linux  (RHEL 7) version 7 or later

Manage your AWS credentials using one of the following methods:

Specify an IAM role when you launch your EC2 instance.

Specify AWS credentials when you configure the agent (see awsAccessKeyId and awsSecretAccessKey).

Edit /etc/sysconfig/aws-kinesis-agent to specify your region and AWS access keys.

If your EC2 instance is in a different AWS account, create an IAM role to provide access to the Kinesis Data Firehose service, and specify that role when you configure the agent (see assumeRoleARN and assumeRoleExternalId). Use one of the previous methods to specify the AWS credentials of a user in the other account who has permission to assume this role.


The IAM role or AWS credentials that you specify must have permission to perform the Kinesis Data Firehose PutRecordBatch operation for the agent to send data to your delivery stream

If you enable CloudWatch monitoring for the agent, permission to perform the CloudWatch PutMetricData operation is also need

By specifying multiple flow configuration settings, you can configure the agent to monitor multiple file directories and send data to multiple streams.

You can specify different endpoints for Kinesis Data Streams and Kinesis Firehose so that your Kinesis stream and Kinesis Firehose delivery stream don’t need to be in the same region.

The agent can pre-process the records parsed from monitored files before sending them to your delivery stream. You can enable this feature by adding the dataProcessingOptions configuration setting to your file flow

The agent supports the following processing options. Because the agent is open-source, you can further develop and extend its processing options.
  -  SINGLELINE  : Converts a multi-line record to a single line record by removing newline characters/leading/trailing spaces
  -  CSVTOJSON 
  -  LOGTOJSON  : Converts from Apache Common Log / Apache Combined Log / Apache Error Log / RFC3164 Syslog to JSON
  - 
