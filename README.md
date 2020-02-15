# AWS Security Monitoring Stack CFT for use with Splunk HEC
CFT that creates CloudWatch Alerts and Events that are sent to both an SNS topic and Kinesis. The Kinesis Firehose Streams are then sent to Splunk via HTTP Event Collector (HEC). This CFT also creates the minimum necessary IAM roles and policies needed.

## Prerequisites and recommendations
### Splunk
1. Kinesis requires Splunk HEC to have a trusted CA-signed certificate. Self-signed certificates are not supported. **Required**
    - This will help: https://answers.splunk.com/answers/462131/securing-http-event-collector.html
2. You will need to have Splunk HEC already configured and the token ready before running this CFT. **Required**
    - This will help: https://docs.splunk.com/Documentation/Splunk/8.0.2/Data/UsetheHTTPEventCollector#Configure_HTTP_Event_Collector_on_Splunk_Enterprise
3. You should install the Splunk Add-on for Kinesis Firehose. Recommended
    - Splunkbase link: https://splunkbase.splunk.com/app/3719/

### AWS
1. You will need the lambda processor function created, zipped, and placed in an accessible S3 bucket. **Required**
    - You can create it from the AWS Blueprint "kinesis-firehose-cloudwatch-logs-processor" or use the ZIP in the repo :)
    - The function is to long to be created within the CFT
2. You will need the name of a Log Group used by a Cloudtrail as end point. You can create this or use an existing one. **Required**
    - This will help: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/send-cloudtrail-events-to-cloudwatch-logs.html#send-cloudtrail-events-to-cloudwatch-logs-console-create-log-group


## Parameters used in this CFT
- SplunkHECEndpoint - URL and port to Splunk HEC endpoint
- SplunkHECToken - Splunk HEC token
- SplunkS3BackupMode - Decide if you want only failed events or all events written to S3
    - AllowedValues: 
        - FailedEventsOnly
        - AllEvents
- SplunkS3ErrorOutputPrefix - Prefix for errors only written to S3
- CloudtrailLogGroup - A CloudTrail Log group
- LambdaProcessorS3Bucket - Accessible S3 bucket that contains the Lambda ZIP
- LambdaProcessorFunctionZIP - The name of the Lambda ZIP
- SNSSubscriptionEmailAlerts - E-mail address for the CloudWatch alerts, can be same as events
- SNSSubscriptionEmailEvents - E-mail address for the CloudWatch events, can be same as alerts

## Roles and Policies
There are 3 roles and policies created by this CFT. I've provided the tl;dr, review the actual CFT for more detail.
- KinesisFirehoseS3Role
    - This allows Kinesis to write to the S3 bucket
- CloudWatchFirehoseRole
    - This allows CloudWatch Events and subscription filter to write to the Kinesis Firehose Stream
- LambdaProcessorRole
    - Allow the Lambda to write to the log stream and the Kinesis processor to use the Lambda function

## SNS Topics: 
 - SecurityMonitoringTopicAlerts - SNS Topic for the Alerts
 - SecurityMonitoringTopicEvents - SNS Topic for the Events

## CloudWatch Alarms/Events:
- AlarmName: failed_console_logins - triggers if there are AWS Management Console authentication failures
- AlarmName: vpc_changes - triggers when changes are made to a VPC
- AlarmName: igw_changes - triggers when changes are made to an Internet Gateway in a VPC
- AlarmName: vpc_routetable_changes - triggers when changes are made to a VPC's Route Table
- AlarmName: unauthorized_api_calls - triggers if Multiple unauthorized actions or logins attempted
- AlarmName: root_account_login - triggers if a root user uses the account
- AlarmName: no_mfa_console_logins - triggers if there is a Management Console sign-in without MFA
- AlarmName: iam_user_changes - triggers when changes are made to IAM users
- EventName: detect-config-changes - detects changes to AWS Config
- EventName: detect-cloudtrail-changes - detects changes to CloudTrail configutations
- EventName: detect-iam-policy-changes - detects IAM policy changes
- EventName: detect-s3-bucket-policy-changes - detects changes to S3 bucket policies
- EventName: detect-network-changes - detects changes to network configuration
- EventName: detect-kms-cmk-operations - detects KMS Customer Master Key (CMK)
- EventName: detect-security-group-changes - detects changes to security groups
- EventName: detect-network-acl-changes - detects changes to network ACLs

**Notes:**
 - If you add additonal CloudWatch Alarms/Metrics, you will need to add the metric filter pattern to the 
 subscription filter (SubscriptionFilterFirehoseProcessed)as well.