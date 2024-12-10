# Serverless Web Application
## Description
A Serverless Web App for Math Operations
This project demonstrates a serverless web application built using AWS Amplify to perform math functions through an interactive user interface. It utilizes various AWS services to achieve a scalable and cost-effective solution.
![Serverless Web App drawio](https://github.com/user-attachments/assets/5df3d30f-9b44-4139-9ae0-3936b51c9f52)

## Features

•	Serverless architecture powered by AWS Amplify
•	RESTful API management with API Gateway
•	Backend logic execution with AWS Lambda functions
•	Data storage using Amazon DynamoDB
•	Continuous deployment facilitated by AWS Amplify
•	Secure access control with AWS IAM
## Prerequisites
•	An active AWS account
•	AWS Amplify CLI installed globally
•	Basic understanding of AWS concepts

## Getting Started

## Setup Amplify
1.	Create a New Amplify Application: 
o	Navigate to the AWS Amplify service in the AWS Management Console.
o	Click "Create an app" and provide a name (e.g., FunWithMath) and a branch name (e.g., dev).
![image](https://github.com/user-attachments/assets/a095db31-0806-4fe0-b044-f015c0e88d38)
2.	Deploy the Frontend: 
•	In the "App Setup" section, select "Frontend" and choose a hosting method (e.g., drag and drop your zipped HTML file named "index.zip").
•	Confirm the upload and deployment of your frontend code
![image](https://github.com/user-attachments/assets/177abacc-ac33-41db-9230-da4606d00d86)
the file should be uploaded as index.zip instead of as indicated in the image.
The user interface portion of this project is now live and available!
![image](https://github.com/user-attachments/assets/91c33733-3843-4837-9f44-63bc057369c2)

## Setting Up Lambda Function
1.	Create a Lambda Function: 
o	Navigate to the AWS Lambda service in the console.
o	Click "Create function" and choose "Author from scratch".
o	Assign a name (e.g., FunWithMath) and select the latest Python version for runtime.
![image](https://github.com/user-attachments/assets/d15f3e0a-4207-4b31-8200-4b1ba1c56424)
2.	Implement the Lambda Function: 
o	Paste the following code into the code editor:
# import the JSON utility package
import json
# import the Python math library
import math

# define the handler function that the Lambda service will use an entry point
def lambda_handler(event, context):
# extract the two numbers from the Lambda service's event object
    # extract the two numbers from the Lambda service's event object
    firstValue = int(event['firstValue'])
    secondValue = int(event['secondValue'])
    
    # perform multiplication
    mathResult = firstValue * secondValue
    # return a properly formatted JSON object
    return {
    'statusCode': 200,
    'body': json.dumps('Your result is ' + str(mathResult))
    }
3.	Deploy the Lambda Function: 
o	Click the "Deploy" button to activate your Lambda function.
![image](https://github.com/user-attachments/assets/e01def20-474f-4386-9177-51aa5e1269ee)
Then the next step is to test the code.
![image](https://github.com/user-attachments/assets/629214a0-cdcf-4043-8e36-3047a67d0400)
Setting Up API Gateway
To create a public endpoint for our Lambda function, we'll use the API Gateway service:
•	Navigate to the API Gateway service and create a new "REST API."
![image](https://github.com/user-attachments/assets/f5f25428-1c81-42cc-91fb-7423db9e18f5)
•	Under "Resources," select the forward slash (/) and click "Create Method."
•	Set the HTTP method to "POST" as we'll be sending data to the Lambda function.
![image](https://github.com/user-attachments/assets/35fe207f-7d93-443a-a226-1d98425b50d7)
•	Choose "Lambda Function" as the integration type and select your specific function (e.g., mathFunction). Leave the rest of the settings unchanged. Click "Create Method."
![image](https://github.com/user-attachments/assets/c98af708-f968-4a64-b8ac-8ab6d28edda0)

Next Steps:
•	We'll configure CORS (Cross-Origin Resource Sharing) to allow your Amplify frontend to access the API.
•	We'll create a stage and deploy our API.
![image](https://github.com/user-attachments/assets/23664293-0292-4d63-b378-22a26aee3300)
![image](https://github.com/user-attachments/assets/01c96622-13b2-4659-a9f7-04fc33d4738f)
•	We'll create a stage and deploy our API.
![image](https://github.com/user-attachments/assets/26072889-1303-471f-837e-15f04a0fd91e)
•	Finally, we'll test the API call using the generated invoke URL
![image](https://github.com/user-attachments/assets/185018aa-7b1d-4284-b38f-e7e96b4327de)
Now we can trigger our lambda function through an API call
## Integrating DynamoDB for Result Storage
This section outlines how to store calculation results in a DynamoDB database and retrieve them in the user interface.
1. Create a DynamoDB Table
•	Navigate to the DynamoDB service and create a new table.
•	Name your table appropriately (e.g., FunWithMathResults).
•	Define the table schema with attributes to store relevant data. Here's a suggestion: 
o	UserID (String): A unique identifier for the calculation result.
![image](https://github.com/user-attachments/assets/00899da6-fe45-4bd6-b722-394b43a4ca92)
2. Configure IAM Policy for Lambda Function
•	Go to your Lambda function and navigate to "Configuration" -> "Permissions."
•	Click on the existing role name or create a new role if needed.
•	Under "Add permissions," select "Create an inline policy."
![image](https://github.com/user-attachments/assets/1bc581d4-6f0f-4613-941a-e02096314d20)
•	Use the following JSON policy template, replacing <your-table-arn> with the actual ARN of your DynamoDB table:
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:DeleteItem",
                "dynamodb:GetItem",
                "dynamodb:Scan",
                "dynamodb:Query",
                "dynamodb:UpdateItem"
            ],
            "Resource": "YOUR-TABLE-ARN"
        }
    ]
}
![image](https://github.com/user-attachments/assets/90d2a1a6-d279-4022-8091-37526c0ff455)
•	Name the policy and click "Create policy."
3. Update Lambda Function Code
•	Modify your Lambda function's Python code to interact with DynamoDB:
# Import the JSON utility package
import json

# Import the Python math library
import math

# Import the AWS SDK (for Python the package name is boto3)
import boto3

# Import datetime and zoneinfo for timezone handling
from datetime import datetime
from zoneinfo import ZoneInfo

# Create a DynamoDB object using the AWS SDK
dynamodb = boto3.resource('dynamodb')

# Use the DynamoDB object to select our table
table = dynamodb.Table('FunWithMathDatabase')

# Define the handler function that the Lambda service will use as an entry point
def lambda_handler(event, context):
    # Extract the two numbers from the Lambda service's event object
    firstValue = int(event['firstValue'])
    secondValue = int(event['secondValue'])

    # Perform multiplication instead of exponentiation
    mathResult = firstValue * secondValue

    # Get the current time in SAST
    now = datetime.now(ZoneInfo("Africa/Johannesburg")).strftime("%a, %d %b %Y %H:%M:%S %Z")

    # Write result and time to the DynamoDB table using the object we instantiated and save response in a variable
    response = table.put_item(
        Item={
            'UserID': str(mathResult),
            'LatestGreetingTime': now
        }
    )

    # Return a properly formatted JSON object
    return {
        'statusCode': 200,
        'body': json.dumps('Your result is ' + str(mathResult))
    }
    4. Update Frontend Code (index.html)
•	Replace the placeholder YOUR-API-URL in your index.html file with the actual invoke URL obtained from API Gateway.
In the section that looks like this
// make API call with parameters and use promises to get response
            fetch("YOUR-API-URL", requestOptions)
            .then(response => response.text())
            .then(result => alert(JSON.parse(result).body))
            .catch(error => console.log('error', error));
5. Redeploy the Application
•	Navigate back to your Amplify project.
•	Select "Deploy updates" and upload the updated index.html file.
![image](https://github.com/user-attachments/assets/0c8c138a-db3c-43ce-9586-8662a7e51bff)
Now when you click on the domain link it should open up your app in another tab as it did before.
Give it a try! Put some numbers into the text box and click calculate.
Here is an example showing the results of 3 x 5.
![image](https://github.com/user-attachments/assets/b43026fa-8ddc-4268-bf50-bed7061f0709)
