# Building-a-Real-Time-Monitoring-Pipeline-for-Event-Logs

Scenario:

You are responsible for monitoring security and compliance on a cloud account. Your task is to set up a monitoring system that detects specific events in real-time from activity logs. Whenever a relevant event is found, an alert should be triggered.


Goal:

1 - Enable CloudTrail to capture all API events.

2 - Set up secure storage for activity logs in your account.

3 - Trigger an alerting process whenever a new log file is stored.

4 - Set up a queue to manage log processing tasks.

5 - Deploy a function that:

    a - Processes new log files.
    
    b - Scans for specific security events (defined in a JSON file).
    
    c - Sends an alert to a SNS topic if the event is part of the monitored events.

    d - Stores relevant event details in a DynamoDB table.
    
 
NOTE!
The solution should be deployed using IaC.


Methodology:

	1.	AWS::CloudTrail::Trail

	    •	IsLogging: Enables or disables the trail logging.

	    •	IncludeGlobalServiceEvents: Captures global events, such as IAM actions.

	    •	IsMultiRegionTrail: Ensures the trail is available in all regions.

	    •	EnableLogFileValidation: Enables validation for the integrity of log files.

	    •	EventSelectors: Configures the type of events to capture, such as read/write events for S3 objects and Lambda functions.

	2.	S3 Bucket

	    •	Stores the CloudTrail log files.

	    •	Configured with a bucket policy to allow CloudTrail to write logs securely.

	
    3.	S3 Bucket Policy

	    •	Grants CloudTrail permissions to write logs to the bucket while blocking public access.
        
        •	Updated the bucket policy to allow the Lambda function to read the configuration file.

    4. 	LogAlertTopic

	    •	An SNS Topic that will receive notifications when new log files are stored.

	5.	TrailBucket.NotificationConfiguration

	    •	Configures S3 to send notifications to the SNS Topic for the s3:ObjectCreated:* event.
        
	6.	LogAlertSubscription

	    •	Subscribes an email address to the SNS Topic. Users will receive email notifications for new log events.

	7.	TrailBucketPolicy

	    •	Grants SNS permission to access the bucket notifications.

    8. LogProcessingQueue

	    •	Added an Amazon SQS queue for processing log tasks.

	9.	Bucket Notification to SQS

	    •	Configured S3 to send notifications to the SQS queue when new objects are created.

	10.	Queue Policies

	    •	Added an SQS queue policy (LogProcessingQueuePolicy) to allow S3 to send messages to the queue securely.

    11. DynamoDB Table:

	    •	Created a SecurityEventsTable to store processed events.

	    •	Added a primary key (EventId) to uniquely identify each event.

	12.	Lambda Function:

	    •	Extended the function to save security events (ConsoleLogin, CreateUser) into DynamoDB.

	    •	Added dynamodb permissions to the Lambda execution role.

        •	Fetches the configuration JSON file dynamically from the S3 bucket.

	    •	Reads and monitors the event names from the events.json file.


    13. Configuration File:

	    •	Added an events.json file to define the list of monitored events.
	
 

WORKFLOW:


	1.	S3 Bucket (TrailBucket):

        •	Logs trigger the Lambda function when created.

	2.	Lambda Function (LogProcessorLambda):

	    •	Reads the log file, filters for specific events, and:

        •	The Lambda function fetches the events.json file.

	    •	Filters log entries based on the event names in the JSON file.

        •	Publishes alerts to SNS.

	    •	Saves matching events to DynamoDB and sends alerts via SNS.

	3.	DynamoDB Table (SecurityEventsTable):

	    •	Stores processed security events for further analysis or reporting.

    4. The events.json file specifies which CloudTrail events to monitor.




Deployment steps:

aws cloudformation create-stack \
  --stack-name EnableCloudTrailStack \
  --template-body file://main.yaml \
  --parameters ParameterKey=TrailBucketName,ParameterValue=my-trail-bucket \
                 ParameterKey=AlertEmail,ParameterValue=myemail@email.com \
  --capabilities CAPABILITY_NAMED_IAM

  aws s3 cp events.json s3://<Your-Bucket-Name>/config/monitored_events.json


Testing the Setup

  1.	Ensure the JSON file is uploaded to the correct location:

 aws s3 ls s3://<Your-Bucket-Name>/config/

  2. Trigger the Lambda function by uploading a log file to the S3 bucket.

  3. Verify:

	•	The Lambda function reads the JSON file from the bucket.

	•	Monitored events are correctly processed and stored in DynamoDB.

	•	Alerts are sent via SNS for detected events.

   4. Query DynamoDB for captured events

   aws dynamodb scan --table-name SecurityEvents
   