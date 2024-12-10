# Serverless Web Application
## Description
A Serverless Web App for Math Operations
This project demonstrates a serverless web application built using AWS Amplify to perform math functions through an interactive user interface. It utilizes various AWS services to achieve a scalable and cost-effective solution.

![image](https://github.com/user-attachments/assets/719a804f-4831-4c1d-a116-6910ea326005)


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
- Navigate to the AWS Amplify service in the AWS Management Console.
- Click "Create an app" and provide a name (e.g., FunWithMath) and a branch name (e.g., dev).

![image](https://github.com/user-attachments/assets/8c9ab9f0-6976-4611-835e-0fab999e3524)

2.	Deploy the Frontend:
- In the "App Setup" section, select "Frontend" and choose a hosting method (e.g., drag and drop your zipped HTML file named "index.zip").
- Confirm the upload and deployment of your frontend code.

![image](https://github.com/user-attachments/assets/812e47cd-9744-4a02-9cb1-6a2143f6e38a)

the file should be uploaded as index.zip instead of as indicated in the image.
The user interface portion of this project is now live and available!.

![image](https://github.com/user-attachments/assets/89972dbb-ae1c-4720-88b9-2fa7dd4583e6)

## Setting Up Lambda Function
1.	Create a Lambda Function:
- Navigate to the AWS Lambda service in the console.
- Click "Create function" and choose "Author from scratch".
- Assign a name (e.g., FunWithMath) and select the latest Python version for runtime.

![image](https://github.com/user-attachments/assets/fae500ac-d5ef-4cb6-82a2-e7d60f2160a3)

2.	Implement the Lambda Function:
- Paste the following code into the code editor:
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
- Click the "Deploy" button to activate your Lambda function.

![image](https://github.com/user-attachments/assets/07fdb76b-3220-4418-9353-89190b749d25)

Then the next step is to test the code.

![image](https://github.com/user-attachments/assets/a79cbd37-9925-4962-92d1-3bdce0a87370)

## Setting Up API Gateway
To create a public endpoint for our Lambda function, we'll use the API Gateway service:
- Navigate to the API Gateway service and create a new "REST API.".

![image](https://github.com/user-attachments/assets/9b7e0211-4941-4e2a-b453-8a87c592fc79)

- Under "Resources," select the forward slash (/) and click "Create Method."
- Set the HTTP method to "POST" as we'll be sending data to the Lambda function.

![image](https://github.com/user-attachments/assets/c723dbb2-7e0b-424f-a070-8e6a2e9ea665)

- Choose "Lambda Function" as the integration type and select your specific function (e.g., mathFunction). Leave the rest of the settings unchanged. Click "Create Method.".

![image](https://github.com/user-attachments/assets/a3c6453d-8117-4aa1-9286-068bd35ddf03)

Next Steps:.
- We'll configure CORS (Cross-Origin Resource Sharing) to allow your Amplify frontend to access the API.
- We'll create a stage and deploy our API.

![image](https://github.com/user-attachments/assets/1ee245b5-8d45-4266-b394-be878f10fafc)

![image](https://github.com/user-attachments/assets/c8ef01cd-c9bb-4c2c-b555-1da82f539abf)

- We'll create a stage and deploy our API.

![image](https://github.com/user-attachments/assets/bf4ca14b-c5bb-4a34-bce7-f03d5d73a31d)

- Finally, we'll test the API call using the generated invoke URL.

![image](https://github.com/user-attachments/assets/58187312-9f3d-489a-9983-eae0e2cc4ecd)

Now we can trigger our lambda function through an API call
## Integrating DynamoDB for Result Storage
This section outlines how to store calculation results in a DynamoDB database and retrieve them in the user interface.
1. Create a DynamoDB Table
-Navigate to the DynamoDB service and create a new table.
- Name your table appropriately (e.g., FunWithMathResults).
- Define the table schema with attributes to store relevant data. Here's a suggestion:.
- UserID (String): A unique identifier for the calculation result.

![image](https://github.com/user-attachments/assets/1f295d60-2eaf-4d05-8c3d-3d43dd9e2f91)

2. Configure IAM Policy for Lambda Function
- Go to your Lambda function and navigate to "Configuration" -> "Permissions.".
- Click on the existing role name or create a new role if needed.
- Under "Add permissions," select "Create an inline policy.".

![image](https://github.com/user-attachments/assets/fca053ce-d070-49dd-a85d-214e6a1cc7ee)

- Use the following JSON policy template, replacing <your-table-arn> with the actual ARN of your DynamoDB table:
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

![image](https://github.com/user-attachments/assets/119b44b7-4e75-459e-8d48-62a2bc5d8e54)

- Name the policy and click "Create policy.".
3. Update Lambda Function Code.
- Modify your Lambda function's Python code to interact with DynamoDB:.
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
- Replace the placeholder YOUR-API-URL in your index.html file with the actual invoke URL obtained from API Gateway.
In the section that looks like this.
```
// make API call with parameters and use promises to get response
            fetch("YOUR-API-URL", requestOptions)
            .then(response => response.text())
            .then(result => alert(JSON.parse(result).body))
            .catch(error => console.log('error', error));
```
5. Redeploy the Application.
- Navigate back to your Amplify project.
- Select "Deploy updates" and upload the updated index.html file.

![image](https://github.com/user-attachments/assets/fd0fada8-d530-4d29-9cea-4a3e8934a3be)

Now when you click on the domain link it should open up your app in another tab as it did before.
Give it a try! Put some numbers into the text box and click calculate.
Here is an example showing the results of 3 x 5.

![image](https://github.com/user-attachments/assets/d20198b1-5ea3-4ff6-ae39-e1dfcb59c1fb)

