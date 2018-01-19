# Using the Lambda Preprocessing Feature<a name="lambda-preprocessing"></a>

If the data in your stream needs format conversion, transformation, enrichment, or filtering, you can preprocess the data using an AWS Lambda function before your application SQL code executes, or before your application creates a schema from your data stream\. 

Using a Lambda function for preprocessing records is useful in the following scenarios:

+ Transforming records from other formats \(such as KPL or GZIP\) into formats that Kinesis Data Analytics can analyze\. Kinesis Data Analytics currently supports JSON or CSV data formats\.

+ Expanding data into a format that is more accessible for operations such as aggregation or anomaly detection\. For instance, if several data values are stored together in a string, you can expand the data into separate columns\.

+ Data enrichment with other AWS services, such as extrapolation or error correction\.

+ Applying complex string transformation to record fields\.

+ Data filtering for cleaning up the data\.

## Using a Lambda Function for Preprocessing Records<a name="lambda-preprocessing-use"></a>

When creating your Kinesis Data Analytics application, you enable Lambda preprocessing in the **Connect to a Source** page\.

**To use a Lambda function to preprocess records in a Kinesis Data Analytics application**

1. On the **Connect to a Source** page, choose **Enabled** in the **Record pre\-processing with AWS Lambda** section\.

1. To use a Lambda function that you have already created, choose the function from the **Lambda function** drop\-down list\.

1. To create a new Lambda function from one of the [[ERROR] BAD/MISSING LINK TEXT](#lambda-preprocessing-templates), choose the template from the drop\-down list, and choose the **View <template name> in Lambda** link to edit the function\.

1. To create a new Lambda function, choose **Create new**\. For information about creating a Lambda function, see [Create a HelloWorld Lambda Function and Explore the Console](http://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html)\.

1. Choose the version of the Lambda function to use\. To use the latest version, choose **$LATEST**\.

When you choose or create a Lambda function for record preprocessing, the records are preprocessed before your application SQL code executes, or your application generates a schema from the records\.

## Lambda Preprocessing Permissions<a name="lambda-preprocessing-policy"></a>

To use Lambda preprocessing, the application's IAM role requires the following permissions policy:

```
     {
       "Sid": "UseLambdaFunction",
       "Effect": "Allow",
       "Action": [
           "lambda:InvokeFunction",
           "lambda:GetFunctionConfiguration"
       ],
       "Resource": "<FunctionARN>"
   }
```

For more information about adding permissions policies, see [Authentication and Access Control for Amazon Kinesis Data Analytics](authentication-and-access-control.md)\.

## Lambda Preprocessing Metrics<a name="lambda-preprocessing-metrics"></a>

You can monitor the number of Lambda invocations, bytes processed, successes and failures, etc\. using Amazon CloudWatch\. For information about CloudWatch metrics that are emitted by Kinesis Data Analytics Lambda preprocessing, see [Amazon Kinesis Analytics Metrics](http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aka-metricscollected.html)\.

## Lambda Preprocessing Templates<a name="lambda-preprocessing-templates"></a>

Amazon Kinesis Data Analytics provides the following templates for Lambda functions for preprocessing records in a stream:


| Lambda Blueprint | Language and version | Description | 
| --- | --- | --- | 
| General Kinesis Data Analytics Input Processing  | Node\.js 6\.10 |  A Kinesis Data Analytics record pre\-processor that receives JSON or CSV records as input and then returns them with a processing status\. Use this processor as a starting point for custom transformation logic\.  | 
| General Kinesis Analytics Input Processing  | Python 2\.7 |  A Kinesis Data Analytics record pre\-processor that receives JSON or CSV records as input and then returns them with a processing status\. Use this processor as a starting point for custom transformation logic\.  | 
| Compressed Input Processing | Node\.js 6\.10 | A Kinesis Data Analytics record processor that receives compressed \(GZIP or Deflate compressed\) JSON or CSV records as input and returns decompressed records with a processing status\.  | 
| KPL Input Processing | Python 2\.7 | A Kinesis Data Analytics record processor that receives Kinesis Producer Library \(KPL\) aggregates of JSON or CSV records as input and returns disaggregated records with a processing status\.  | 

## Data Preprocessing Event Input Data Model/ Record Response Model<a name="lambda-preprocessing-data-model"></a>

To preprocess records, your Lambda function must be compliant with the required event input data and record response models\. 

### Event Input Data Model<a name="lambda-preprocessing-request-model"></a>

Kinesis Data Analytics continuously reads data from your Kinesis stream or Kinesis Data Firehose delivery stream\. For each batch of records it retrieves, the service manages how each batch gets passed to your Lambda function\. Your function receives a list of records as input\. Within your function, you iterate through the list and apply your business logic to accomplish your preprocessing requirements \(such as data format conversion or enrichment\)\. 

The input model to your preprocessing function varies slightly, depending on whether the data was received from a Kinesis stream or a Kinesis Data Firehose delivery stream\. 

If the source is a Kinesis Data Firehose delivery stream, the event input data model is as follows:


| Field | Description | 
| --- | --- | 
| Field | Description | 
| --- | --- | 
| Field | Description | 
| --- | --- | 
| invocationId | The Lambda invocation Id \(random GUID\)\. | 
| applicationArn | Kinesis Data Analytics application Amazon Resource Name \(ARN\) | 
| streamArn | Delivery stream ARN | 
| Records [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kinesisanalytics/latest/dev/lambda-preprocessing.html)  | 
| recordId | record ID \(random GUID\) | 
| kinesisFirehoseRecordMetadata |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kinesisanalytics/latest/dev/lambda-preprocessing.html)  | 
| data | Base64\-encoded source record payload | 
| approximateArrivalTimestamp | Delivery stream record approximate arrival time | 

If the source is a Kinesis stream, the event input data model is as follows:

**Kinesis Streams Request Data Model**


| Field | Description | 
| --- | --- | 
| Field | Description | 
| --- | --- | 
| Field | Description | 
| --- | --- | 
| invocationId | The Lambda invocation Id \(random GUID\)\. | 
| applicationArn | Kinesis Data Analytics application ARN | 
| streamArn | Delivery stream ARN | 
| Records [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kinesisanalytics/latest/dev/lambda-preprocessing.html)  | 
| recordId | record ID based off of Kinesis record sequence number | 
| kinesisStreamRecordMetadata |  [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kinesisanalytics/latest/dev/lambda-preprocessing.html)  | 
| data | Base64\-encoded source record payload | 
| sequenceNumber | Sequence number from the Kinesis stream record | 
| partitionKey | Partition key from the Kinesis stream record | 
| shardId | ShardId from the Kinesis stream record | 
| approximateArrivalTimestamp | Delivery stream record approximate arrival time | 

### Record Response Model<a name="lambda-preprocessing-response-model"></a>

All records returned from your Lambda preprocessing function \(with record IDs\) that are sent to the Lambda function must be returned\. They must contain the following parameters, or Kinesis Data Analytics rejects them and treats it as a data preprocessing failure\. The data payload part of the record can be transformed to accomplish preprocessing requirements\.

**Response Data Model**


| Field | Description | 
| --- | --- | 
| recordId | The record ID is passed from Kinesis Data Analytics to Lambda during the invocation\. The transformed record must contain the same record ID\. Any mismatch between the ID of the original record and the ID of the transformed record is treated as a data preprocessing failure\. | 
| result | The status of the data transformation of the record\. The possible values are: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/kinesisanalytics/latest/dev/lambda-preprocessing.html)  | 
| data | The transformed data payload, after base64\-encoding\. Each data payload can contain multiple JSON documents if the application ingestion data format is JSON, or can contain multiple CSV rows \(with a row delimiter specified in each row\) if the application ingestion data format is CSV\. The Kinesis Data Analytics service successfully parses and processes data with either multiple JSON documents or CSV rows within the same data payload\.  | 

## Common Data Preprocessing Failures<a name="lambda-preprocessing-failures"></a>

The following are common reasons why preprocessing can fail\.

+ Not all records \(with record IDs\) in a batch that are sent to the Lambda function are returned back to the Kinesis Data Analytics service\. 

+ The response is missing either the record ID, status, or data payload field\. The data payload field is optional for a `Dropped` or `ProcessingFailed` record\.

+ The Lambda function timeouts are not sufficient to preprocess the data\.

+ The Lambda function response exceeds the response limits imposed by the AWS Lambda service\.

In the case of data preprocessing failures, the Kinesis Data Analytics service continues to retry Lambda invocations on the same set of records until successful\. You can monitor the following CloudWatch metrics to gain insight into failures\.

+ Kinesis Data Analytics Application `MillisBehindLatest`: Indicates how far behind an application is reading from the streaming source\. 

+ Kinesis Data Analytics Application `InputPreprocessing` CloudWatch metrics: Indicates the number of successes and failures, among other statistics\. For more information, see [Amazon Kinesis Analytics Metrics](http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aka-metricscollected.html)\.

+ AWS Lambda function CloudWatch metrics and logs\.

## For Further Information<a name="lambda-preprocessing-further-information"></a>

For further information, including architectural information and a sample Lambda function to preprocess stream records, see [Pre\-Processing Data in Kinesis Analytics with AWS Lambda](https://aws.amazon.com/blogs/big-data/preprocessing-data-in-amazon-kinesis-analytics-with-aws-lambda/)\. 