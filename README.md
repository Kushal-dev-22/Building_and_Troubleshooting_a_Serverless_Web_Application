
# Building and Troubleshooting a Serverless Web Application

This project demonstrates the end-to-end deployment of a serverless 3-tier web application using AWS Lambda, Amazon API Gateway (via Function URL), Amazon S3, and Amazon DynamoDB. Following the initial deployment, the project shifts focus to operational excellence by integrating observability using AWS X-Ray and troubleshooting real-world failures such as runtime dependency errors and IAM permission issues.

## Objective

-   **Provision Backend**: Manually provision a serverless backend using AWS CloudShell and the CLI.
    
-   **Host Frontend**: Deploy a client-side application on Amazon S3 with static website hosting.
    
-   **Implement Observability**: Enable distributed tracing using AWS X-Ray to visualize service maps.
    
-   **Troubleshoot Dependencies**: Diagnose and resolve "Module Not Found" errors using AWS Lambda Layers.
    
-   **Simulate Failures**: Analyze permission failures in a distributed system.
    

## Prerequisites

-   An active AWS Account with Administrator access.
    
-   Basic familiarity with the AWS Management Console and AWS CLI.
    
-   Files from the repository: `dynamodb_commands.txt`, `items.json`, `lambda_function.py`, `index.html`, `error.html`, `cookie.jpg`, `layer.zip`, and `lambda_function_xray.py`.
    

----------

## Step 1: Login to AWS Management Console

1.  Open your web browser.
    
2.  Navigate to the AWS Management Console login page.
    
3.  Log in using your AWS credentials.
    

## Step 2: Initialize Database via CloudShell

1.  Click the **CloudShell** icon in the top navigation bar to open the browser-based CLI.
    
2.  Clone the repository to the CloudShell environment:bash
    
    git clone [https://github.com/ACloudGuru-Resources/hands-on-aws-troubleshooting.git](https://www.google.com/search?q=https://github.com/ACloudGuru-Resources/hands-on-aws-troubleshooting.git)
    
3.  Navigate to the project directory:
    
    Bash
    
    ```
    cd hands-on-aws-troubleshooting/Building_and_Troubleshooting_a_Serverless_Web_Application
    
    ```
    
4.  Open the `dynamodb_commands.txt` file to view the table creation syntax.
    
5.  Execute the command from the text file to create the DynamoDB table (e.g., `Fortunes`).
    

## Step 3: Populate Database Data

1.  Verify the `items.json` file is present in your directory.
    
2.  Execute the batch-write command to populate the table with initial data:
    
    Bash
    
    ```
    aws dynamodb batch-write-item --request-items file://items.json
    
    ```
    

## Step 4: Create IAM Role for Lambda

1.  Navigate to **IAM** → **Roles** → **Create role**.
    
2.  Select **AWS service** and choose **Lambda** as the trusted entity.
    
3.  Click **Next: Permissions**.
    
4.  Create a custom policy (or attach an existing one) that grants `dynamodb:Scan` and `dynamodb:GetItem` permissions on the table created in Step 2.
    
5.  Name the role (e.g., `LambdaDynamoReadRole`) and create it.
    

## Step 5: Create Lambda Function

1.  Navigate to the **AWS Lambda** console.
    
2.  Click **Create function**.
    
3.  Select **Author from scratch**.
    
4.  Name the function (e.g., `FortuneAppFunction`).
    
5.  Select **Python 3.9** as the runtime.
    
6.  Under **Permissions**, select **Use an existing role** and choose the role created in Step 4.
    
7.  Copy the code from `lambda_function.py` in the repo and paste it into the **Code Source** editor.
    
8.  Click **Deploy**.
    

## Step 6: Configure Function URL and Test

1.  Click **Test** in the Lambda console to verify the function executes without errors.
    
2.  Navigate to the **Configuration** tab → **Function URL**.
    
3.  Click **Create function URL**.
    
4.  Select **NONE** for Auth type (to allow public access for this lab) and check **Configure CORS**.
    
5.  Save the configuration and copy the generated **Function URL**.
    

## Step 7: Connect Frontend to Backend

1.  Open the `index.html` file on your local machine.
    
2.  Locate the JavaScript variable (often named `apiEndpoint` or similar) inside the script tag.
    
3.  Paste the **Function URL** you copied in Step 6 into the variable value.
    
4.  Save the file.
    

## Step 8: Create S3 Bucket and Upload Assets

1.  Navigate to the **Amazon S3** console.
    
2.  Click **Create bucket** and give it a unique name.
    
3.  Uncheck **Block all public access** and acknowledge the warning (required for public website hosting).
    
4.  Create the bucket.
    
5.  Upload the modified `index.html`, `error.html`, and `cookie.jpg`.
    
6.  Select the uploaded files, go to **Actions** -> **Make public using ACL** (or modify the Bucket Policy to allow `s3:GetObject` for `Principal: *` if ACLs are disabled).
    

## Step 9: Enable Static Website Hosting

1.  Go to the **Properties** tab of your S3 bucket.
    
2.  Scroll down to **Static website hosting** and click **Edit**.
    
3.  Select **Enable**.
    
4.  Enter `index.html` for the **Index document** and `error.html` for the **Error document**.
    
5.  Save changes.
    

## Step 10: Access the Application

1.  In the S3 bucket properties, copy the **Bucket website endpoint** URL.
    
2.  Paste the URL into a new browser tab.
    
3.  Verify the website loads and displays the content (including the `cookie.jpg` image) and retrieves data from the API.
    

## Step 11: Enable AWS X-Ray Tracing

1.  Return to your Lambda function in the AWS Console.
    
2.  Go to **Configuration** → **Monitoring and operations tools**.
    
3.  Click **Edit**.
    
4.  Toggle **Active tracing** (under AWS X-Ray) to **On**.
    
5.  Click **Save**.
    

## Step 12: Configure Lambda Layer

1.  In the Lambda console, select **Layers** from the left menu -> **Create layer**.
    
2.  Name the layer `XRaySDK`.
    
3.  Upload the `layer.zip` file from the repository.
    
4.  Choose **Python 3.9** as the compatible runtime and create the layer.
    
5.  Go back to your Lambda function, scroll to the **Layers** section, click **Add a layer**, and attach the custom layer you just created.
    

## Step 13: Update Code for Observability

1.  Open the `lambda_function_xray.py` file from the repository.
    
2.  Extract the import statements and the X-Ray patch code (e.g., `from aws_xray_sdk.core import xray_recorder...`).
    
3.  Update your Lambda function code by pasting these lines into the appropriate sections (imports at the top, patch code before the handler).
    
4.  **Deploy** the changes.
    

## Step 14: Analyze X-Ray Traces

1.  Trigger the Lambda function again by refreshing your S3 website multiple times.
    
2.  Navigate to **CloudWatch** → **X-Ray traces** (or the X-Ray console).
    
3.  Open the **Service Map** to visualize the request flow from the Client → Lambda → DynamoDB.
    
4.  Verify that the nodes are green (successful).
    

## Step 15: Simulate and Diagnose Failure

1.  Go to the IAM role created in Step 4.
    
2.  Remove/Detach the DynamoDB permission policy.
    
3.  Refresh the S3 website (it should now fail to load data).
    
4.  Return to the **X-Ray traces** service map.
    
5.  Observe that the Lambda node or the link to DynamoDB is now showing an error (Orange/Red), confirming that the permission change successfully broke the application logic.
    

----------

## Conclusion

In this lab, we successfully built a serverless web application and navigated a real-world troubleshooting scenario. By initially deploying code that lacked required dependencies (`aws_xray_sdk`), we simulated a runtime failure and resolved it by creating and attaching an AWS Lambda Layer. Finally, we utilized AWS X-Ray to visualize the application's performance and architecture, deliberately breaking permissions to observe how failures are reported in a distributed system.