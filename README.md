# 2.14
Wong Choon Yee
assignment 2.14


- Does SNS guarantee exactly once delivery to subscribers? 

No, Amazon SNS (Simple Notification Service) does not guarantee "exactly once" delivery to subscribers. More than that instead, it offers at least once delivery, which means that SNS will attempt to deliver the message to subscribers, and while it will try to avoid duplication, there is a possibility that a message could be delivered more than once in certain situations (e.g., network retries).

Delivery Guarantees in AWS SNS:
1.	At Least Once Delivery: SNS guarantees that each message will be delivered at least once to each subscriber, but in case of network failures or retries, duplicate messages may be delivered.
2.	Message Ordering: SNS does not guarantee that messages will be delivered in the order they were sent. If the order is important, you would need to handle message ordering on the subscriber's end.
3.	Failure Handling: If a subscriber is temporarily unavailable (e.g., if a Lambda function fails or an HTTP/S endpoint is down), SNS will retry delivery according to a backoff strategy. For some protocols (like email or SMS), SNS might also retry delivery.
4.	Deduplication (for FIFO topics): If you use SNS with a FIFO (First In, First Out) topic, you can enable message deduplication, which can help reduce the chances of duplicate deliveries, but it still does not provide exactly-once delivery.
If you need strict exactly once message delivery guarantees, you would typically need to implement that at the subscriber level by keeping track of message IDs or employing idempotent processing logic.



- What is the purpose of the Dead-letter Queue (DLQ)? This a feature available to SQS/SNS/EventBridge. 

A Dead-letter Queue (DLQ) is a specialized queue used in messaging systems (including AWS services like SNS, SQS, and Lambda) to capture and store messages that could not be successfully processed. The primary purpose of a DLQ is to provide a way to handle and inspect failed messages for troubleshooting and debugging purposes.
Key Purposes of a Dead-letter Queue:
1.	Capture Failed Messages:
•	If a message cannot be processed by the main queue (due to issues like timeouts, invalid message formats, or other errors), it is sent to the DLQ for later inspection.
•	This helps avoid losing messages that couldn't be delivered or processed successfully.
2.	Message Retention:
•	Failed messages are retained in the DLQ so that developers and administrators can investigate the root cause of the failure, such as malformed messages, configuration issues, or problems with the processing logic.
3.	Troubleshooting and Debugging:
•	By analyzing the messages in the DLQ, you can identify patterns of failures or specific types of errors that are causing problems.
•	It helps in fixing bugs or improving the reliability of the system, especially when messages fail consistently.
4.	Message Reprocessing:
•	After resolving the issue causing the failure, messages in the DLQ can be reprocessed by manually or automatically retrying them, either back to the original processing queue or through a different processing flow.
5.	Separate Failure Handling Logic:
•	A DLQ can be configured with its own set of rules, such as different retention periods or dead-lettering thresholds, to better control how failures are handled.
•	This allows for a separation of concerns between successful processing and error handling.
Common Use Cases:
•	SNS to SQS: If SNS sends messages to an SQS queue, and those messages fail to be delivered to subscribers (like Lambda functions or HTTP/S endpoints), the messages can be placed in the DLQ for further investigation.
•	Lambda Functions: If a Lambda function triggered by an event fails multiple times, the event data can be sent to a DLQ for analysis.
•	SQS: If an SQS queue is configured with a DLQ, messages that cannot be processed by the consumers (due to errors or retries) are sent to the DLQ.
How It Works in AWS:
•	SNS to DLQ: When SNS cannot deliver a message to a subscriber after a defined number of retries, it can send the failed message to a DLQ (usually an SQS queue).
•	SQS to DLQ: If a consumer (e.g., Lambda) fails to process messages from an SQS queue a certain number of times, those messages can be sent to a DLQ.
•	Lambda to DLQ: If a Lambda function fails to process an event multiple times, the event is placed in a DLQ (often an SQS queue) for investigation.
Benefits of Using DLQs:
•	Message Reliability: Ensure that no message is lost even if it can't be processed initially.
•	Error Isolation: Prevent errors from affecting the normal processing flow by isolating failed messages.
•	Improved Debugging: By reviewing the messages in the DLQ, you can identify recurring issues and improve your system.



- How would you enable a notification to your email when messages are added to the DLQ? 

1.	Create an SNS Topic for Notifications:
•	In the SNS console, create a new SNS topic that will send email notifications when messages are added to the DLQ.
•	To do this:
•	Go to the SNS console.
•	Click on Create topic.
•	Select Standard or FIFO (whichever fits your use case).
•	Enter a name for the topic (e.g., DLQ-Notifications).
•	Click Create topic.

2.	Subscribe to the SNS Topic:
•	After creating the SNS topic, you need to subscribe to it so that email notifications are sent to your inbox.
•	In the SNS console, click on the topic you just created.
•	Under Subscriptions, click Create subscription.
•	Select Email as the protocol.
•	Enter your email address in the Endpoint field.
•	Click Create subscription.
•	You will receive a confirmation email. You must confirm the subscription by clicking on the confirmation link in the email.

3.	Create a CloudWatch Alarm:
•	Set up a CloudWatch Alarm that will trigger whenever messages are added to your DLQ.
•	You can monitor the DLQ (such as an SQS queue) for the ApproximateNumberOfMessagesVisible metric to detect when messages are being added to the queue.
•	Go to the CloudWatch console.
•	Click on Alarms in the left-hand navigation pane, then click Create Alarm.
•	Select SQS as the metric namespace.
•	Choose the DLQ from the list of available SQS queues.
•	Select the metric ApproximateNumberOfMessagesVisible (this shows how many messages are in the queue).
•	Set the threshold for the alarm. For example, you could set an alarm to trigger when the number of messages in the DLQ is greater than 0, or if the number exceeds a certain threshold for a specific period (e.g., 5 messages for 5 minutes).
•	Choose the SNS topic (the one you created earlier) as the notification action for the alarm.

4.	Configure the Alarm:
•	Once you have set the condition (such as the threshold for the number of messages in the DLQ), proceed to configure the notification actions.
•	Select the SNS topic you created earlier to notify via email when the alarm is triggered.

5.	Test the Setup:
•	To verify that the system works, you can manually add a message to the DLQ (e.g., by causing a failure in your processing system).
•	If the DLQ accumulates messages, the CloudWatch alarm should trigger and send an email notification to the subscribed address.



Detailed Steps for CloudWatch Alarm Configuration:
1.	In the CloudWatch Console, navigate to Alarms and click Create Alarm.
2.	Choose Select metric and search for your SQS queue under the SQS namespace.
3.	Select the metric ApproximateNumberOfMessagesVisible and click select metric.
4.	Set the threshold condition. For example:
•	Whenever: Greater than or equal to
•	Threshold type: Static or Anomaly detection.
•	Threshold value: 1 (or another threshold depending on how many messages you consider "noteworthy").
5.	Set the actions: Choose Send a notification to SNS topic and select the SNS topic created earlier.
6.	Finally, configure other settings like the name of the alarm and click Create alarm.
Now, every time the threshold is crossed (e.g., when new messages are added to the DLQ), you will receive an email notification.


Benefits of This Setup:
•	Real-time alerts: You are immediately notified when messages are added to the DLQ, allowing you to take action quickly.
•	Automatic monitoring: The system automatically tracks the number of messages in the DLQ and notifies you when there is an issue.

