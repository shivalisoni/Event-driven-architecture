# Event-driven-architecture
Building event-driven architectures on AWS

This is from an AWS hosted event, I tried it myself from their website to better understand the event driven architecture.

Setup :

1. Launch the AWS CloudFormation template :
   AWS CloudFormation allowed me to codify the infrastructure.
   1) They had a list of regions to choose to create the template from, I chose the N.Virginia region. Create a cloudformation stack.
 
2. Configure the Event Generator:
   
   Once the stack creation has been completed navigate to the stack Outputs tab. There will be a list of values necessary to o configure the Event Generator.
   1) Right-click the url in the EventGeneratorConfigurationUrl output, opening a new browser window.
   2) This opens the Event Generator website and pre-populate the modal dialog box with values for a Cognito User Pool and Cognito Identity Pool provisioned in your        AWS account. Click Configure Cognito User Pool to view the sign-in page.
   3)On the Sign In page add your the CognitoUsername and CognitoPassword from the CloudFormation Stack Output page.
   4) After successful sign in, you verify that the Cognito user has been configured and that your account id has been pre-populated in the AWS Account ID field.

3. Event driven with SNS: 
    
   This is a pub sub implementation. An Amazon SNS topic will be used to publish events to an Amazon SQS queue subscriber.
    
    1) Create an SQS queue named OrdersQueue by selecting a standard queue.
    2) From the list of queues, choose the OrdersQueue queue to which to subscribe the Orders Amazon SNS topic. The topic was created by the cloudformation template used, but can be created manually as well.
    3) From Queue Actions, select Subscribe Queue to SNS Topic and select the Orders topic.
    4) To verify the result of the subscription, the Event Generator will be used to publish to the topic and then view the message that the topic sends to the queue.
    5)Select the SNS tab in the Event Generator . Choose the Orders topic from the list of Topics then publish the message to the Orders SNS topic.
    6)In SQS from the list of queues, choose the OrdersQueue queue. The OrdersQueue queue now shows the value 1 in the Messages Available column in the list. This means the Orders SNS topic has successfully sent the message to the queue.
    7)Select Send and receive messages. On details screen, choose Poll for messages.
    8)Before the visibility timeout expires, select the message that you want to delete and then choose Delete 1 Message.



    Message filtering :
    
    1) Create an Amazon SQS queue, called Orders-EU.
    2) On details screen, select Subscribe to Amazon SNS topic , then select the Orders topic to which to subscribe the Orders-EU queue, and then choose Subscribe.
    3) In SNS, choose the Orders topic . Select the Orders-EU subscription and then choose Edit.
    4) On the Edit subscription page, expand the Subscription filter policy section. In the JSON editor field, paste the following filter policy into the JSON editor.
                 {
           "location": [
             "eu-west"
           ]
         }
      
     5) Open the Event Generator for SNS .If not pre-selected choose the Orders topic from the list of Topics.
     6) Choose Add attribute to add a message attributes. 
          Message 1 : Non present attribute : For the attribute Type, leave the selection as String. For the attribute Name, enter category. For the                             attribute Value, enter books, then publish.
          
          Message 2 : Non matching message attribute : For the attribute Type, leave the selection as String. For the attribute Name, enter location. For the attribute           Value, enter us-west.
     
          Message 3 : Matching message attribute : For the attribute Type, leave the selection as String. For the attribute Name, enter location. For the attribute               Value, enter eu-west
          
          
       7) In SQS, the Orders queue will now have 3 messages, as it does not have the filter and the Orders-EU queue only shows 1 message delivered. This is because                only Message 3 had the matching message attribute key and value (location = eu-west) to satisfy the subscription filter.

       
        Error Handling : Client side error handling by using DLQ:
        
        1) In SNS, from the list of topics, choose the Orders topic and copy the ARN.
        2) In SQS, create a nw queue named OrdersTopicDLQ.
        3) From queue details page, navigate to Access policy tab.
        4) Click Edit button and on the details page, append below policy statement to already existing one with correct ARN of Order SNS topic and OrdersTopicDLQ ARN.

               {
                 "Sid": "topic-subscription-sns",
                 "Effect": "Allow",
                 "Principal": {
                   "AWS": "*"
                 },
                 "Action": "SQS:SendMessage",
                 "Resource": "OrdersTopicDLQ_ARN",
                 "Condition": {
                   "ArnLike": {
                     "aws:SourceArn": "SNS_ARN"
                   }
                 }
                }
                
           5) From the cloudformation template, a lambda function called DodgyFunction should have been created . In lambda, select that function, in the Lambda                     function Designer, select Add trigger.
           6) Enter SNS in the Lambda function Trigger filter and select the SNS trigger.
           7) Select Orders from the SNS topic list and add.
           8) Select the SNS tab in the Event Generator.
           9) The Orders topic is pre-selected from the list of Topics.Choose Publish to publish the message to the Orders SNS topic.
           10) In lambda, select the DodgyFunction, choose the Monitoring tab and note that the Lambda function was invoked successfully one time.
           11) In SNS, choose the Orders topic, and select the DodgyFunction from the list of Subscriptions and edit.
           12) Click on the Redrive policy (dead-letter queue) toggle button, select the OrdersTopicDLQ ARN in the Dead-letter queue field, and choose Save                            changes.
           13) Now, go to lambda  from the console and delete the DodgyFunction lamda function.
           14) Select the SNS tab in the Event Generator. Choose Publish to publish the message to the Orders SNS topic.
           
            The Orders queue shows 2 messages delivered, because the Orders subscription has no dead-letter queue and received both messages published from the Event               Generator.

            The OrdersTopicDLQ queue shows 1 message delivered, because the DodgyFunction subscription has a dead-letter queue and received the failed message delivery             after the Lambda function was deleted.



