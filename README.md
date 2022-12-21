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
