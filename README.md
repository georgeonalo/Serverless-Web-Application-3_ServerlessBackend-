# Module 3: Serverless Service Backend

In this module you'll use AWS Lambda and Amazon DynamoDB to build a backend process for handling requests from your web application. The browser application that you deployed in the first module allows users to request that a unicorn be sent to a location of their choice. In order to fulfill those requests, the JavaScript running in the browser will need to invoke a service running in the cloud.

You'll implement a Lambda function that will be invoked each time a user requests a unicorn. The function will select a unicorn from the fleet, record the request in a DynamoDB table and then respond to the front-end application with details about the unicorn being dispatched.

![image](https://user-images.githubusercontent.com/115881685/208951009-2f56f64f-f7fc-46f8-918d-4d10c30fbbe1.png)

The function is invoked from the browser using Amazon API Gateway. You'll implement that connection in the next module. For this module you'll just test your function in isolation.

## 1. Create an Amazon DynamoDB Table

Use the Amazon DynamoDB console to create a new DynamoDB table. Call your table Rides and give it a partition key called RideId with type String. The table name and partition key are case sensitive. Make sure you use the exact IDs provided. Use the defaults for all other settings.

After you've created the table, note the ARN for use in the next step.

### ✅ Step-by-step directions

1. Go to the Amazon DynamoDB Console

2. Choose Create table.

3. Enter Rides for the Table name. This field is case sensitive.

4. Enter RideId for the Partition key and select String for the key type. This field is case sensitive.

5. Check the Use default settings box and choose Create.

![image](https://user-images.githubusercontent.com/115881685/208951654-82efe4e3-c482-4db2-8b58-c7b0c4d33496.png)

6. Scroll to the bottom of the Overview section of your new table and note the ARN. You will use this in the next section.

#### 2. Create an IAM Role for Your Lambda function

##### Background

Every Lambda function has an IAM role associated with it. This role defines what other AWS services the function is allowed to interact with. For the purposes of this workshop, you'll need to create an IAM role that grants your Lambda function permission to write logs to Amazon CloudWatch Logs and access to write items to your DynamoDB table.

##### High-Level Instructions

Use the IAM console to create a new role. Name it WildRydesLambda and select AWS Lambda for the role type. You'll need to attach policies that grant your function permissions to write to Amazon CloudWatch Logs and put items to your DynamoDB table.

Attach the managed policy called AWSLambdaBasicExecutionRole to this role to grant the necessary CloudWatch Logs permissions. Also, create a custom inline policy for your role that allows the ddb:PutItem action for the table you created in the previous section.

##### ✅ Step-by-step directions

1. Go to the AWS IAM Console

2. Select Roles in the left navigation bar and then choose Create role.

3. Select Lambda for the role type from the AWS service group, then click Next: Permissions Note: Selecting a role type automatically creates a trust policy for your role that allows AWS services to assume this role on your behalf. If you were creating this role using the CLI, AWS CloudFormation or another mechanism, you would specify a trust policy directly.

4. Begin typing AWSLambdaBasicExecutionRole in the Filter text box and check the box next to that role.

5. Click Next: Tags. Add any tags that you wish.

6. Click Next: Review.
 
7. Enter WildRydesLambda for the Role name.

8. Choose Create role.

Next you need to add permissions to the role so that it can access your DynamoDB table.

✅ Step-by-step directions

1. While in the IAM Console on the roles page type WildRydesLambda into the filter box on the Roles page and choose the role you just created.

2. On the Permissions tab, choose the Add inline policy link in the lower right corner to create a new inline policy.

![image](https://user-images.githubusercontent.com/115881685/208954691-859a3c28-82f2-463d-a7b6-4a45a61b954d.png)

3. Select Choose a service.

4. Begin typing DynamoDB into the search box labeled Find a service and select DynamoDB when it appears.

![image](https://user-images.githubusercontent.com/115881685/208954917-aecdb1d9-ed66-4bfc-b53a-3d5902ae6708.png)

5. Choose Select actions.

6. Begin typing PutItem into the search box labeled Filter actions and check the box next to PutItem when it appears.

7. Select the Resources section.

8. With the Specific option selected, choose the Add ARN link in the table section.

9. Paste the ARN of the table you created in the previous section in the Specify ARN for table field, and choose Add.

10. Choose Review Policy.

11. Enter DynamoDBWriteAccess for the policy name and choose Create policy.

![image](https://user-images.githubusercontent.com/115881685/208955601-aecbb373-61bf-4e2a-b699-d06fa7cbb45f.png)

##### 3. Create a Lambda Function for Handling Requests

##### Background

AWS Lambda will run your code in response to events such as an HTTP request. In this step you'll build the core function that will process API requests from the web application to dispatch a unicorn. In the next module you'll use Amazon API Gateway to create a RESTful API that will expose an HTTP endpoint that can be invoked from your users' browsers. You'll then connect the Lambda function you create in this step to that API in order to create a fully functional backend for your web application.

##### High-Level Instructions

Use the AWS Lambda console to create a new Lambda function called RequestUnicorn that will process the API requests. Use the provided requestUnicorn.js: ([https://github.com/georgeonalo/Serverless-Web-Application-requestUnicorn.js]) example implementation for your function code. Just copy and paste from that file into the AWS Lambda console's editor.

Make sure to configure your function to use the WildRydesLambda IAM role you created in the previous section.

##### ✅ Step-by-step directions

1. Go to the AWS Lambda

2. Click Create function.

3. Keep the default Author from scratch card selected.

4. Enter RequestUnicorn in the Name field.

5. Select Node.js 8.10 for the Runtime.

6. Expand Choose or create an execution role under Permissions.

7. Ensure Choose an existing role is selected from the Role dropdown.

8. Select WildRydesLambda from the Existing Role dropdown.

![image](https://user-images.githubusercontent.com/115881685/208958749-b39b4330-881d-40fa-8914-f5441bf3206d.png)

9. Click on Create function.

10. Scroll down to the Function code section and replace the existing code in the index.js code editor with the contents of requestUnicorn.js.

![image](https://user-images.githubusercontent.com/115881685/208959450-f7b566ef-fe2a-4636-baa7-64a7e2df3714.png)

11. Click "Save" in the upper right corner of the page.

##### Implementation Validation

For this module you will test the function that you built using the AWS Lambda console. In the next module you will add a REST API with API Gateway so you can invoke your function from the browser-based application that you deployed in the first module.

##### ✅ Step-by-step directions

1. From the main edit screen for your function, select Configure test events from the the Select a test event... dropdown.

![image](https://user-images.githubusercontent.com/115881685/208959875-e374add7-b5e8-45dc-8d75-6cb12e226f53.png)

2. Keep Create new test event selected.

3. Enter TestRequestEvent in the Event name field

4. Copy and paste the following test event into the editor:

{

    "path": "/ride",
    
    "httpMethod": "POST",
    
    "headers": {
    
        "Accept": "*/*",
        
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        
        "content-type": "application/json; charset=UTF-8"
        
    },
    
    "queryStringParameters": null,
    
    "pathParameters": null,
    
    "requestContext": {
    
        "authorizer": {
        
            "claims": {
            
                "cognito:username": "the_username"
                
            }
            
        }
        
    },
    
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    
}

![image](https://user-images.githubusercontent.com/115881685/208960697-760f2079-2de1-4d53-a989-17a45e7d9875.png)

5. Click Create.

6. On the main function edit screen click Test with TestRequestEvent selected in the dropdown.

7. Scroll to the top of the page and expand the Details section of the Execution result section.

8. Verify that the execution succeeded and that the function result looks like the following:

{

  "statusCode": 201,
  
  "body": "{\"RideId\":\"1h0zDZ-6KLZaEQCPyqTxeQ\",\"Unicorn\":{\"Name\":\"Shadowfax\",\"Color\":\"White\",\"Gender\":\"Male\"},\"UnicornName\":\"Shadowfax\",\"Eta\":\"30 seconds\",\"Rider\":\"the_username\"}",
  
  "headers": {
  
    "Access-Control-Allow-Origin": "*"
    
  }
  
}

##### ⭐ Recap

🔑 AWS Lambda is a serverless functions as a service product that removes the burden of managing servers to run your applications. You configure a trigger and set the role that the function can use and then can interface with almost anything you want from databases, to datastores, to other services eithe publicly on the internet or in your own Amazon Virtual Private Cloud (VPC). Amazon DynamoDB is a non-relational serverless database that can scale automatically to handle massive amounts of traffic and data without you need manage any servers.

🔧 In this module you've created a DynamoDB table and then a Lambda function to write data into it. This function will be put behind an Amazon API Gateway in the next module which will in turn be connected to your web application to capture the ride details from your users.

##### Next

✅ After you have successfully tested your new function using the Lambda console, you can move on to the next module, RESTful APIs: ([https://github.com/georgeonalo/Serverless-Web-Application-4_RESTfulAPIs-])
