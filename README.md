# Car Insurance Bot
![Architecture](/img/2-chatbot-diagram-simple.png "Architecture")

The diagram demonstrates a Car Insurance Bot solution using AWS serverless services to create a scalable, decoupled and highly available architecture using managed machine learning services.

The message flow starts from a user request through a route configured in an Amazon API Gateway HTTP API, the request is then processed by AWS Lambda.

The AWS Lambda function is responsible for identifying the route and validating the received .json and upon identifying a message request, the lambda will validate the content and create an intent request to Amazon Lex. After processing the intent, Amazon Lex will return to AWS Lambda a response containing the text to be returned to the user.

When the image upload route is triggered, an asynchronous stream is started, which will store the image in Amazon Simple Storage Service (S3) and a message is added to a default queue of Amazon SQS containing the image information to be processed.

Amazon SQS will then trigger the second AWS Lambda function that will take the image content and infer the custom machine learning model that was previously trained on Amazon Rekognition Custom Labels.

After the image analysis is performed, the model will return a json object with the analysis results. The lambda function will then store this result in an Amazon DynamoDB table and the results will be available for querying via an HTTP API route.

Amazon DynamoDB is responsible for storing transactional logs and user information, as well as analysis results.

## Pre-requisites
* AWS CLI
* To run the AWS CloudFormation template that creates the infrastructure, a pre trained Machine Learning Model should be provided and inputed in the **RekognitionModel** field. See how to create an Rekognition Custom Labels [here](https://www.youtube.com/watch?v=AXK3rhe1_FI&ab_channel=AmazonWebServices) and [here](https://www.youtube.com/watch?v=Lx9Y8ZnkHkc&ab_channel=AmazonWebServices).
* You must import the Amazon Lex bot v1 after run CloudFormation, to do it run this following command on terminal:
    - wget {URLGIT}/OctankBot_lexv1.json;
    - zip OctankBot_lexv1 OctankBot_lexv1.json & aws lex-models start-import --resource-type BOT --merge-strategy FAIL_ON_CONFLICT —-payload fileb://./OctankBot_lexv1.zip;
    - After you import you need build and publish(Console):
      - In Amazon Lex v1 console, select your bot:
         ![lex-step-1](/img/1-lex-console.png)
      - Select Build:
         ![lex-step-2](/img/2-lex-console.png)
      - Status of build:
        ![lex-step-3](/img/3-lex-console.png)
      - Now select Publish and set “dev” as alias name:
        ![lex-step-4](/img/4-lex-console.png)
        ![lex-step-5](/img/5-lex-console.png)
      - Now you lex bot is up and running.

## Running
To upload the template using the AWS console:

1. Open *AWS CloudFormation > Stacks > Create Stack > With new resources (standard)*
2. Step 1: Create stack
    1. Prerequisite - Prepare template > Select *Template is ready*
    2. Specify template > **Upload a template file** > upload the .yaml > Click *Next*
3. Step 2: Specify stack details
    - **Stack Name:** CarInsurance
    - **LexBotAlias:** dev
    - **LexBotName:** OctankBot
    - **RekognitionModel:** ARN model
    - Click **Next**
4. Step 3: Configure stack options
    - Leave the default configurations
    - Click **Next**
5. Step 4: Review CarInsurance
    - Advance to Capabilities and Transforms
    - Check all boxes:
    - ![Permission](/img/1-cloudformation.png)
    - Click **Submit**

The provision might take a few minutes to run. 

## Testing
The solution allows the following requests:


1. POST /sendMessage
2. POST /uploadImage
3. GET /getResults


To test the requests and responses of each API call [Postman](https://www.postman.com/) can be used.

API calls breakdown:

1. POST /sendMessage
    - This API call will be responsible for interacting with the Lex chatbot, through intents and responding back to the user with a response (message).
2. POST /uploadImage
    - This API call will upload a base64 binary image to the S3 bucket created for this project. Accepted Content-Types are 'image/png', 'image/jpg', 'image/jpeg'.

1. GET /getResults
   - This API call will be responsible for getting the analysis results done by the Amazon Rekognition Custom Labels model stored in an Amazon DynamoDB table. The request returns to the user a message containing the image label and the confidence rate of the prediction, or in case of null response or error, it will return to the user a message to redo the request.

## Cleaning up

[Deleting a AWS CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-cli-deleting-stack.html)


## Security issue notifications
If you discover a potential security issue in this project we ask that you notify AWS/Amazon Security via our [vulnerability reporting page](http://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public github issue.

See [CONTRIBUTING](https://github.com/aws-samples/aws-car-insurance-bot/blob/main/CONTRIBUTING.md#security-issue-notifications) for more information.

## Licensing

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.