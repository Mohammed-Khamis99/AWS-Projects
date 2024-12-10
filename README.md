# Serverless Web Application
## Description
A Serverless Web App for Math Operations
This project demonstrates a serverless web application built using AWS Amplify to perform math functions through an interactive user interface. It utilizes various AWS services to achieve a scalable and cost-effective solution.

![image](https://github.com/user-attachments/assets/6da32ef2-8aae-42e1-9948-33269dcbdc70)


## Features

- Serverless architecture powered by AWS Amplify.
- RESTful API management with API Gateway.
- Backend logic execution with AWS Lambda functions.
- Data storage using Amazon DynamoDB.
- Continuous deployment facilitated by AWS Amplify.
- Secure access control with AWS IAM.
## Prerequisites
- An active AWS account.
- AWS Amplify CLI installed globally.
- Basic understanding of AWS concepts.

## Getting Started

## Setup Amplify
1.	Create a New Amplify Application:
o	Navigate to the AWS Amplify service in the AWS Management Console.
o	Click "Create an app" and provide a name (e.g., FunWithMath) and a branch name (e.g., dev).

![image](https://github.com/user-attachments/assets/8d371db7-dcfa-4985-94bd-ab2cd44125cb)


2.	Deploy the Frontend:
•	In the "App Setup" section, select "Frontend" and choose a hosting method (e.g., drag and drop your zipped HTML file named "index.zip").
•	Confirm the upload and deployment of your frontend code.

![image](https://github.com/user-attachments/assets/288edd78-cd81-41b7-bc8c-f8b50ca8dccc)

the file should be uploaded as index.zip instead of as indicated in the image.
The user interface portion of this project is now live and available!.

![image](https://github.com/user-attachments/assets/5087426d-ada1-41dc-9f4a-a9d4624634fe)

## Setting Up Lambda Function
1.	Create a Lambda Function: 
o	Navigate to the AWS Lambda service in the console.
o	Click "Create function" and choose "Author from scratch".
o	Assign a name (e.g., FunWithMath) and select the latest Python version for runtime.

![image](https://github.com/user-attachments/assets/7f71f8a1-269d-4b25-916e-e7ae18edd18c)


2.	Implement the Lambda Function: 
o	Paste the following code into the code editor:
```
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
```
3.	Deploy the Lambda Function: 
o	Click the "Deploy" button to activate your Lambda function.

![image](https://github.com/user-attachments/assets/27356aaa-57d2-4555-b536-8a3e79de0d23)

Then the next step is to test the code.

![image](https://github.com/user-attachments/assets/49698645-9317-4013-9ab6-fee46db4a632)

## Setting Up API Gateway
To create a public endpoint for our Lambda function, we'll use the API Gateway service:
•	Navigate to the API Gateway service and create a new "REST API.".

![image](https://github.com/user-attachments/assets/1b67841a-1b5c-4620-b207-3398e0176462)

•	Under "Resources," select the forward slash (/) and click "Create Method."
•	Set the HTTP method to "POST" as we'll be sending data to the Lambda function.

![image](https://github.com/user-attachments/assets/2f533fa0-88dc-4295-9c3f-bc9d7a4b595a)

•	Choose "Lambda Function" as the integration type and select your specific function (e.g., mathFunction). Leave the rest of the settings unchanged. Click "Create Method.".

![image](https://github.com/user-attachments/assets/cc488228-2fea-420f-8af1-53a4263d2b26)


Next Steps:
•	We'll configure CORS (Cross-Origin Resource Sharing) to allow your Amplify frontend to access the API.
•	We'll create a stage and deploy our API.

![image](https://github.com/user-attachments/assets/5934b0e6-ab03-4099-b253-1009b12e077e)

![image](https://github.com/user-attachments/assets/186a2ad0-b2c6-45d3-b9e5-088000022f81)

•	We'll create a stage and deploy our API.

![image](https://github.com/user-attachments/assets/b6077ed4-2ba2-4e49-93b5-d1e32db5b4d0)

•	Finally, we'll test the API call using the generated invoke URL.

![image](https://github.com/user-attachments/assets/adfb66c4-e8c9-4d22-ba66-9cb51ea97c38)

Now we can trigger our lambda function through an API call
## Integrating DynamoDB for Result Storage
This section outlines how to store calculation results in a DynamoDB database and retrieve them in the user interface.
1. Create a DynamoDB Table
•	Navigate to the DynamoDB service and create a new table.
•	Name your table appropriately (e.g., FunWithMathResults).
•	Define the table schema with attributes to store relevant data. Here's a suggestion:.
- UserID (String): A unique identifier for the calculation result.

![image](https://github.com/user-attachments/assets/5b820997-b331-42bf-9a4e-4cd7c9285cd7)

2. Configure IAM Policy for Lambda Function.
•	Go to your Lambda function and navigate to "Configuration" -> "Permissions.".
•	Click on the existing role name or create a new role if needed.
•	Under "Add permissions," select "Create an inline policy.".

![image](https://github.com/user-attachments/assets/b1f87d83-4f44-4d9f-84eb-8ed61da87d5a)

•	Use the following JSON policy template, replacing <your-table-arn> with the actual ARN of your DynamoDB table:
```
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
```

![image](https://github.com/user-attachments/assets/26cff9ac-f3f4-42f6-8b15-a9a5a3df07a5)

•	Name the policy and click "Create policy.".
3. Update Lambda Function Code.
•	Modify your Lambda function's Python code to interact with DynamoDB:.
```python
# Import the JSON utility package
import json

# Import the AWS SDK (for Python the package name is boto3)
import boto3

# Import datetime and zoneinfo for timezone handling
from datetime import datetime
from zoneinfo import ZoneInfo

# Create a DynamoDB resource using the AWS SDK
dynamodb = boto3.resource('dynamodb')

# Use the DynamoDB resource to select our table
table = dynamodb.Table('FunWithMathDatabase')

# Define the handler function that the Lambda service will use as an entry point
def lambda_handler(event, context):
    try:
        # Extract the two numbers from the Lambda service's event object
        first_value = int(event['firstValue'])
        second_value = int(event['secondValue'])

        # Perform multiplication
        math_result = first_value * second_value

        # Get the current time in SAST
        now = datetime.now(ZoneInfo("Africa/Johannesburg")).strftime("%a, %d %b %Y %H:%M:%S %Z")

        # Write result and time to the DynamoDB table
        response = table.put_item(
            Item={
                'UserID': str(math_result),
                'LatestGreetingTime': now
            }
        )

        # Return a properly formatted JSON object
        return {
            'statusCode': 200,
            'body': json.dumps(f'Your result is {math_result}')
        }

    except Exception as e:
        # Handle exceptions and return an error message
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }
   ```
4. Update Frontend Code (index.html).
•	Replace the placeholder YOUR-API-URL in your index.html file with the actual invoke URL obtained from API Gateway.
In the section that looks like this.
```
// make API call with parameters and use promises to get response
            fetch("YOUR-API-URL", requestOptions)
            .then(response => response.text())
            .then(result => alert(JSON.parse(result).body))
            .catch(error => console.log('error', error));
```
5. Redeploy the Application.
•	Navigate back to your Amplify project.
•	Select "Deploy updates" and upload the updated index.html file.

![image](https://github.com/user-attachments/assets/9adaa69f-1ebb-48cf-9f32-2e01939196ad)

Now when you click on the domain link it should open up your app in another tab as it did before.
Give it a try! Put some numbers into the text box and click calculate.
Here is an example showing the results of 3 x 5.

![image](https://github.com/user-attachments/assets/8f86423c-6023-42a2-8203-94f0fff2d022)


