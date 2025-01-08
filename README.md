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

 
