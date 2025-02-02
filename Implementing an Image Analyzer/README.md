# Image Analyzer to Detect Photo Friendliness 📸

A serverless AWS solution to evaluate if an image meets criteria for being "social media-friendly" by analyzing clarity, lighting, and composition using AI.

---

## 🚀 Overview  

This project automates photo quality assessment using **Amazon Rekognition**. Clients upload images via an API, and the system returns feedback on whether the photo is suitable for professional profiles (e.g., LinkedIn). Built with Python, Terraform, and AWS serverless services.

## ✨ Features 

- **AI-Powered Analysis**: Detects blur, lighting issues, facial clarity, and background distractions.  
- **Multi-Format Support**: Accepts `.png` and `.jpeg` files.  
- **Serverless Architecture**: Scalable and cost-efficient with API Gateway, Lambda, and Rekognition.  
- **Infrastructure-as-Code**: Fully deployed using Terraform.  

## 📋 Architecture 

![Image](https://github.com/user-attachments/assets/4e22e680-476a-42e9-9dd7-c62684b51b63)

1. **Client Application**: Sends image via HTTPS to API Gateway.  
2. **API Gateway**: Triggers the Lambda function.  
3. **AWS Lambda (Python)**: Processes the image, queries Rekognition, and parses results.  
4. **Amazon Rekognition**: Returns AI-driven analysis (faces, labels, moderation).  
5. **Response**: JSON feedback indicating "photo-friendly" status or issues detected.  

## ⚙️ Prerequisites  

- AWS account with IAM permissions for Lambda, Rekognition, and API Gateway.  
- Terraform (~> v1.5) and AWS CLI configured.  
- Python 3.9+ for Lambda function.  
- Postman (for testing).

![Image](https://github.com/user-attachments/assets/86600134-c80b-42de-a84e-3a85a4972c96)

![Image](https://github.com/user-attachments/assets/916ada18-a493-4ed9-aa50-dcdad8a6b0d9)

![Image](https://github.com/user-attachments/assets/bc5218cc-c904-468e-b2df-189a1006aee4)

![Image](https://github.com/user-attachments/assets/47f88a56-d77c-48b6-b6f8-2bf8de4cfdff)

![Image](https://github.com/user-attachments/assets/71f25e4c-b5f9-4d0e-b151-c96f89cfd9ac)
