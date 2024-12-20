# Hosting a Static Website using Amazon S3 and CloudFront


Amazon S3 stores your static website files, while CloudFront speeds up delivery by caching content globally. This combination ensures your site loads quickly and securely for users worldwide.
This guide provides step-by-step instructions for hosting a static website on Amazon S3 and accelerating it with CloudFront.

![image](https://github.com/user-attachments/assets/666beffd-7cc1-4e9b-85b8-027fdabc2f4a)

## Prerequisites

Before you begin, ensure you have the following:
- An AWS account
- Basic knowledge of Amazon S3 and CloudFront
- Your static website files (e.g., `index.html`, `portfolio.pdf`)

## Create an S3 Bucket
1. Sign in to the [AWS Management Console] and open the Amazon S3 console.
2. Create a new bucket:
   - Click "Create bucket".

![image](https://github.com/user-attachments/assets/898b6eeb-4c31-4c49-ad33-8cfcf5797f76)

   - Enter a unique bucket name (e.g., `your-portfolio-bucket`).
   - Make the bucket publicly accessible.

![image](https://github.com/user-attachments/assets/fdd6b489-4087-4a30-9594-e695370ec997)

   - Select the appropriate AWS region.
   - Click "Create".

## Enable Static Website Hosting

1. Go to the **Properties** tab of your newly created bucket.
2. Scroll down to the **Static website hosting** section and click "Edit".

![image](https://github.com/user-attachments/assets/b6eba454-dbb4-47ea-9276-ff6298c53524)

3. Select "Enable".
4. Enter `index.html` as the Index document.
5. Leave the Error document field empty or set it to another file if needed.

![image](https://github.com/user-attachments/assets/1d32841a-a6c0-4612-843a-841992cccb5f)

6. Click "Save changes".
## Upload Your Website Files.
1. Click on the "Upload" button in the top-right corner.
2. Select "Add files" and upload your static files (e.g., `index.html`, `portfolio.pdf`).

![image](https://github.com/user-attachments/assets/e0e94498-5aec-4bd3-8840-019af7bbb12a)

## Set Permissions.
1. Go to the **Permissions** tab and click "Bucket Policy".
2. Add the following policy to allow public access:
   ```
   json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::your-portfolio-bucket/*"
       }
     ]
   }```
   
![image](https://github.com/user-attachments/assets/1af68323-6fba-4b47-a418-58a1751266f0)

3. Save the policy.
4. Configure the bucket access control list (ACL) For (Read) Everyone (public access).

![image](https://github.com/user-attachments/assets/cad3c7dd-df6d-4aac-8cf1-76dffa9840c3)

5. Use the bucket website  endpoint to access your website.

![image](https://github.com/user-attachments/assets/25a3579a-a6a3-41e8-bb0d-0111142cbd20)

![image](https://github.com/user-attachments/assets/052fbf48-a26f-4fb8-b5c3-3fb72c244da4)

##  Create a CloudFront Distribution
1.	Open the Amazon CloudFront console.
2.	Click "Create Distribution".

![image](https://github.com/user-attachments/assets/b8a0b277-5e33-4e97-847c-f2eff87a606b)

3.	Configure the distribution settings:
-	Origin Domain Name: Select your S3 bucket's endpoint.
-	Default Cache Behavior Settings: Customize settings as needed.

![image](https://github.com/user-attachments/assets/afe3889d-3888-45e3-a0ed-56fade559287)

-	Viewer Protocol Policy: Choose "Redirect HTTP to HTTPS" or "HTTPS Only".

![image](https://github.com/user-attachments/assets/bb4ccf9f-073b-4d81-9bb1-d52b4f801c37)

-	Click "Create Distribution".
## Update Bucket Policy for CloudFront
1.	Go back to your S3 bucket in the S3 console.
2.	Click on the Permissions tab and then "Bucket Policy".
3.	Add the following policy to restrict access to CloudFront only via Origin Access Control (OAC).

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "StringNotEquals": {
          "aws:Referer": "your-cloudfront-distribution-url"
        }
      }
    }
  ]
}
```

![image](https://github.com/user-attachments/assets/1222f36f-cceb-48e4-af7d-2eeb630866fb)

4.	Ensure the policy allows CloudFront access.
5.	Now if you try to access the website from the bucket website  endpoint the access will be denied. 

![image](https://github.com/user-attachments/assets/eadcc960-7408-48ea-994f-1f147b53a94a)

## Access Your Website
1.	Copy the CloudFront Distribution Domain Name provided in the CloudFront console.

![image](https://github.com/user-attachments/assets/6b49e89f-0202-4384-b69d-d335b6bddebe)

2.	Paste this URL into your browser to view your static website.

![image](https://github.com/user-attachments/assets/08a05f5e-6ad1-498b-a761-8e691635fe7d)








