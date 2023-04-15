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


3. Event-driven with Eventbridge:

    1)Create a custom event bus named Orders.Leave Event archive and Schema discovery disabled, Resource-based policy blank while creating.
    2)Setup Cloudwatch target : 
      1) Select Rules,  from the Event bus dropdown, select the Orders event bus.
      2) Create Rule, add OrdersDevRule as the Name of the rule and select Rule with an event pattern for the Rule type.
      3) Build event pattern, under event source, choose Other(for custom events).
      4) Under Event pattern, further down the screen, enter the following pattern to catch all events from com.aws.orders:
         {
        "source": ["com.aws.orders"]
         }
      5) Select  rule target:

         From the Target dropdown, select CloudWatch log group and name it as /aws/events/orders
     
     3)Now the new rule can be tested :
       1) Go to the Event Generator and Eventbus should be selected to Orders. In the Detail Type add Order Notification
          JSON payload for the Detail Template should be:
          {
             "category": "lab-supplies",
             "value": 415,
             "location": "eu-west"
          }
        
       2) Under the eventbridge tab, click publish.
       3) Open Cloudwatch, choose Log groups in the left navigation and select the /aws/events/orders log group.
       4) Select the Log stream and toggle the log event to verify that you received the event.

