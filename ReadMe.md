---
title: "CRUD API with Authentication"
summary: "Documentation for a serverless CRUD API with authentication, using AWS Lambda, API Gateway, DynamoDB, Cognito, and CodePipeline."
---

# **CRUD API with Authentication - Documentation**

This documentation provides a comprehensive guide to building and deploying a serverless CRUD API using AWS Lambda, API Gateway, DynamoDB, and AWS Cognito for authentication. The deployment process is automated using AWS CodePipeline.

---

## **Table of Contents**
1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Step-by-Step Setup](#step-by-step-setup)
   - [Create a DynamoDB Table](#create-a-dynamodb-table)
   - [Create a Cognito User Pool](#create-a-cognito-user-pool)
   - [Create a Lambda Function](#create-a-lambda-function)
   - [Create an API Gateway](#create-an-api-gateway)
   - [Integrate Cognito with API Gateway](#integrate-cognito-with-api-gateway)
4. [Deploy Using CodePipeline](#deploy-using-codepipeline)
5. [Testing the API](#testing-the-api)
6. [API Endpoints](#api-endpoints)
7. [Code Samples](#code-samples)
8. [Troubleshooting](#troubleshooting)

---

## **1. Architecture Overview**

This system uses the following AWS services:
- **AWS Cognito**: For user authentication and management.
- **AWS API Gateway**: To expose RESTful API endpoints.
- **AWS Lambda**: To handle CRUD operations.
- **AWS DynamoDB**: For storing and retrieving data.
- **AWS CodePipeline**: For continuous deployment of code changes.

---

## **2. Prerequisites**
- AWS Account.
- Installed **AWS CLI** configured with access to your AWS account.
- GitHub repository containing:
  - `main.go` (Lambda code).
  - `buildspec.yml` for CodeBuild.
- Basic knowledge of AWS services.

---

## **3. Step-by-Step Setup**

### **3.1 Create a DynamoDB Table**
1. Go to the [DynamoDB Console](https://console.aws.amazon.com/dynamodb/).
2. Create a table:
   - Table name: `TodoTable`
   - Primary key: `id` (String)
3. Note the **Table Name** for use in Lambda.

---

### **3.2 Create a Cognito User Pool**
1. Navigate to the [Cognito Console](https://console.aws.amazon.com/cognito/).
2. Create a new User Pool:
   - Name: `TodoAppUserPool`.
   - Enable email-based user sign-in.
3. Add an **App Client** without a client secret.
4. Save the **User Pool ID** and **App Client ID**.

---

### **3.3 Create a Lambda Function**
1. Write a **Golang** function for CRUD operations (sample in [Code Samples](#code-samples)).
2. Deploy the function:
   - Runtime: `Go 1.x`.
   - Environment variables:
     - `DYNAMODB_TABLE_NAME`: `TodoTable`.
     - `COGNITO_REGION`: e.g., `us-east-1`.
     - `USER_POOL_ID`: User Pool ID from Cognito.
3. Add **AWS SDK for DynamoDB** to your Go module:
   ```bash
   go get github.com/aws/aws-sdk-go-v2
   ```

---

### **3.4 Create an API Gateway**
1. Go to the [API Gateway Console](https://console.aws.amazon.com/apigateway/).
2. Create a **REST API**:
   - Name: `TodoAPI`.
3. Add Resources and Methods:
   - `/todo`:
     - **POST**: Create a new task.
     - **GET**: Retrieve all tasks.
   - `/todo/{id}`:
     - **PUT**: Update a task.
     - **DELETE**: Delete a task.
4. Integrate each method with your Lambda function.

---

### **3.5 Integrate Cognito with API Gateway**
1. Navigate to the **Authorizers** section in API Gateway.
2. Create a Cognito Authorizer:
   - Name: `CognitoAuth`.
   - Select your **User Pool**.
3. Assign the Authorizer to API Methods:
   - Under Method Request → **Authorization**, choose `CognitoAuth`.

---

## **4. Deploy Using CodePipeline**

### **4.1 Set Up S3 for Artifacts**
- Create an S3 bucket (e.g., `my-lambda-artifacts`).

### **4.2 Add `buildspec.yml` to the Repository**
Add a `buildspec.yml` file for CodeBuild to compile and package the Lambda function:
```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      golang: 1.20
  build:
    commands:
      - echo Building the Go binary...
      - GOOS=linux GOARCH=amd64 go build -o main main.go
  post_build:
    commands:
      - echo Packaging the binary into a ZIP file...
      - zip function.zip main
artifacts:
  files:
    - function.zip
```

### **4.3 Create a CodePipeline**
1. Go to the [CodePipeline Console](https://console.aws.amazon.com/codepipeline/).
2. Create a pipeline with these stages:
   - **Source**: GitHub repository (main branch).
   - **Build**: Use CodeBuild with `buildspec.yml`.
   - **Deploy**: Deploy the `function.zip` to your Lambda function.

---

## **5. Testing the API**
1. Obtain a JWT token:
   - Sign up and log in using Cognito.
   - Use the token from the response for API requests.
2. Make API calls:
   - Include the token in the `Authorization` header:
     ```bash
     curl -H "Authorization: Bearer <JWT_TOKEN>" -X GET https://<API_GATEWAY_URL>/todo
     ```

---

## **6. API Endpoints**

| Method | Endpoint         | Description            |
|--------|------------------|------------------------|
| POST   | `/todo`          | Create a new task.     |
| GET    | `/todo`          | Retrieve all tasks.    |
| PUT    | `/todo/{id}`     | Update a task.         |
| DELETE | `/todo/{id}`     | Delete a task.         |

---

## **7. Code Samples**

### **Lambda Handler**
Refer to the earlier example in this conversation for a detailed Golang implementation of token validation and CRUD operations.

---

## **8. Troubleshooting**

### **Common Issues**
1. **Unauthorized Error**:
   - Ensure the Cognito Authorizer is attached to the API methods.
   - Verify the token is valid and not expired.

2. **Lambda Errors**:
   - Check Lambda environment variables.
   - Review CloudWatch logs for detailed error messages.

3. **Pipeline Fails**:
   - Verify the `buildspec.yml` is correctly configured.
   - Ensure IAM roles have sufficient permissions.

---

Let me know if you need further assistance with this documentation!
