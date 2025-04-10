

## ⚡ **Introduction to AWS Lambda & Serverless Computing**

---

### ✅ What is Serverless Computing?

Serverless doesn’t mean “no servers” — it means:
- **You don’t manage the servers.**
- AWS **automatically provisions, scales, and manages** the infrastructure.
- You only pay for the **compute time your code actually uses**.

---

### ✅ What is AWS Lambda?

**AWS Lambda** is a **compute service** that runs your code **in response to events** and **automatically manages** the compute resources.

You upload your code, and Lambda takes care of:
- Scaling
- Fault tolerance
- Availability
- Infrastructure

---

### ✅ Common Use Cases:
- Running backend logic (e.g., user signup handler)
- File processing (e.g., resize an image uploaded to S3)
- Real-time stream processing (e.g., Kinesis)
- Scheduled tasks (like cron jobs)
- Chatbots, APIs (with API Gateway)

---

## 🛠️ Hands-On: Create Your First AWS Lambda Function

---

### 🧪 Step 1: Go to Lambda Console
1. Open AWS Console → Go to **AWS Lambda** service
2. Click **"Create function"**

---

### ⚙️ Step 2: Configure the Function
1. Choose **Author from scratch**
2. Function name: `helloLambda`
3. Runtime: Choose `Python 3.x`, `Node.js`, or `Java` (Let’s use Python for simplicity)
4. Permissions:
   - Choose **“Create a new role with basic Lambda permissions”**

Click **Create Function**

---

### 💻 Step 3: Add Your Code
In the **inline editor**, replace the code with:

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

Click **Deploy**

---

### 🚀 Step 4: Test Your Lambda Function
1. Click **Test**
2. Create a new test event (use default settings)
3. Click **Test** again

✅ You should see a `200 OK` response with `"Hello from Lambda!"`

---

## 🔄 Lambda Triggers: Understand Event-Driven Architecture

---

### ✅ What Triggers a Lambda Function?

Lambda can be triggered by many AWS services:

| Service            | Trigger Example                                 |
|--------------------|--------------------------------------------------|
| **S3**             | File upload to bucket (e.g., image processing)  |
| **API Gateway**    | HTTP requests (build a REST API)                |
| **CloudWatch**     | Scheduled tasks (like cron jobs)                |
| **DynamoDB/Kinesis** | Stream data processing                        |
| **SNS/SQS**        | Messaging and alerts                            |

---

### 🧪 Example: Trigger Lambda from an S3 Upload

1. Create an S3 bucket
2. In Lambda → **Add Trigger** → Select **S3**
3. Choose the bucket and event type (e.g., `ObjectCreated`)
4. Save

Now, **uploading a file to S3 will trigger the Lambda function**.

---

## ✅ Summary

| Topic                         | Covered                        |
|------------------------------|--------------------------------|
| Serverless Computing Basics  | ✅                              |
| Created Lambda Function       | ✅ (`helloLambda`)              |
| Tested Function               | ✅                              |
| Explored Triggers             | ✅ (event-driven architecture)  |

---


## 🚀 **Use Case 1: Build a Serverless REST API with API Gateway + Lambda**

---

### 🔧 Goal: Create a REST API endpoint that triggers a Lambda function

---

### 🧩 Step 1: Create Lambda Function

Use this simple Python function:

```python
def lambda_handler(event, context):
    name = event.get("queryStringParameters", {}).get("name", "World")
    return {
        "statusCode": 200,
        "body": f"Hello, {name} from Lambda!"
    }
```

✅ Deploy it in the Lambda console.

---

### 🧩 Step 2: Create API Gateway

1. Go to **API Gateway → Create API**
2. Choose **HTTP API**
3. Click **Build**
4. Add integration → **Lambda Function**
5. Select the Lambda function you created earlier
6. Add route: `GET /hello`
7. Click **Next → Deploy**

🎯 You’ll get a **public URL endpoint** like:

```
https://abc123.execute-api.us-east-1.amazonaws.com/hello?name=John
```

✅ It returns: `"Hello, John from Lambda!"`

---

## 📦 **Use Case 2: S3 Trigger → Lambda for File Processing**

---

### 🧩 Step 1: Create an S3 Bucket

- Go to **S3 → Create bucket**
- Enable versioning if you want
- Allow public access **off**

---

### 🧩 Step 2: Create Lambda Function

Use this example Python code:

```python
def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        file = record['s3']['object']['key']
        print(f"New file uploaded: {file} in {bucket}")
```

Give it **S3 read permissions** in the Lambda’s IAM role.

---

### 🧩 Step 3: Add S3 as Trigger

1. In Lambda → Add Trigger → Select **S3**
2. Choose the bucket
3. Event type: **Object Created (All)**
4. Click Add

✅ Now, uploading a file to S3 triggers your Lambda function.

---

## ⏰ **Use Case 3: Scheduled Job Using CloudWatch Events**

---

### 🧩 Step 1: Create Lambda Function

Use this Python code:

```python
import datetime

def lambda_handler(event, context):
    now = datetime.datetime.now()
    print(f"Scheduled run at: {now}")
```

---

### 🧩 Step 2: Create CloudWatch Event Rule

1. Go to **CloudWatch → Rules (EventBridge)**
2. Click **Create rule**
3. Choose **Schedule**
4. Use a cron expression like:

```
cron(0/5 * * * ? *)   → runs every 5 minutes
```

5. Add **target**: your Lambda function
6. Create rule

✅ Lambda now runs on a schedule!



## ⚙️ **Part 1: AWS Lambda + API Gateway (CloudFormation)**

This CloudFormation template will:

✅ Create a Lambda function  
✅ Create an API Gateway endpoint to invoke it  
✅ Set up necessary IAM roles and permissions

---

### 📄 `lambda-api.yaml` (CloudFormation Template)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Deploy Lambda with API Gateway

Resources:
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: LambdaBasicExecutionRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

  HelloWorldFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: HelloWorldLambda
      Handler: index.lambda_handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Runtime: python3.9
      Code:
        ZipFile: |
          def lambda_handler(event, context):
              name = event.get("queryStringParameters", {}).get("name", "World")
              return {
                  "statusCode": 200,
                  "body": f"Hello, {name} from Lambda!"
              }

  ApiGateway:
    Type: AWS::ApiGatewayV2::Api
    Properties:
      Name: HelloWorldApi
      ProtocolType: HTTP
      Target: !GetAtt HelloWorldFunction.Arn

  LambdaPermission:
    Type: AWS::Lambda::Permission
    Properties:
      Action: lambda:InvokeFunction
      FunctionName: !Ref HelloWorldFunction
      Principal: apigateway.amazonaws.com

  ApiIntegration:
    Type: AWS::ApiGatewayV2::Integration
    Properties:
      ApiId: !Ref ApiGateway
      IntegrationType: AWS_PROXY
      IntegrationUri: !Sub arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${HelloWorldFunction.Arn}/invocations
      PayloadFormatVersion: '2.0'

  ApiRoute:
    Type: AWS::ApiGatewayV2::Route
    Properties:
      ApiId: !Ref ApiGateway
      RouteKey: 'GET /hello'
      Target: !Sub integrations/${ApiIntegration}

  ApiDeployment:
    Type: AWS::ApiGatewayV2::Deployment
    DependsOn:
      - ApiRoute
    Properties:
      ApiId: !Ref ApiGateway

  ApiStage:
    Type: AWS::ApiGatewayV2::Stage
    Properties:
      ApiId: !Ref ApiGateway
      StageName: prod
      DeploymentId: !Ref ApiDeployment

Outputs:
  ApiUrl:
    Description: "Invoke this URL to trigger Lambda"
    Value: !Sub https://${ApiGateway}.execute-api.${AWS::Region}.amazonaws.com/prod/hello
```

---

### ✅ Deploying It:

```bash
aws cloudformation deploy \
  --template-file lambda-api.yaml \
  --stack-name LambdaWithApi \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## ⚙️ **Part 2: Terraform Equivalent**

---

### 📁 Directory Structure:

```
terraform-lambda-api/
├── main.tf
├── lambda_function.py
```

---

### 🐍 `lambda_function.py` (inline Python code)

```python
def lambda_handler(event, context):
    name = event.get("queryStringParameters", {}).get("name", "World")
    return {
        "statusCode": 200,
        "body": f"Hello, {name} from Lambda!"
    }
```

---

### 🔧 `main.tf` (Terraform Code)

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_iam_role" "lambda_exec" {
  name = "lambda_exec_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Principal = { Service = "lambda.amazonaws.com" }
      Effect    = "Allow"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "basic_exec" {
  role       = aws_iam_role.lambda_exec.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

data "archive_file" "lambda_zip" {
  type        = "zip"
  source_file = "${path.module}/lambda_function.py"
  output_path = "${path.module}/lambda_function.zip"
}

resource "aws_lambda_function" "hello_lambda" {
  function_name = "HelloLambda"
  role          = aws_iam_role.lambda_exec.arn
  handler       = "lambda_function.lambda_handler"
  runtime       = "python3.9"
  filename      = data.archive_file.lambda_zip.output_path
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256
}

resource "aws_apigatewayv2_api" "api" {
  name          = "hello-api"
  protocol_type = "HTTP"
}

resource "aws_lambda_permission" "apigw" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.hello_lambda.arn
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.api.execution_arn}/*/*"
}

resource "aws_apigatewayv2_integration" "integration" {
  api_id           = aws_apigatewayv2_api.api.id
  integration_type = "AWS_PROXY"
  integration_uri  = aws_lambda_function.hello_lambda.invoke_arn
  payload_format_version = "2.0"
}

resource "aws_apigatewayv2_route" "route" {
  api_id    = aws_apigatewayv2_api.api.id
  route_key = "GET /hello"
  target    = "integrations/${aws_apigatewayv2_integration.integration.id}"
}

resource "aws_apigatewayv2_stage" "stage" {
  api_id      = aws_apigatewayv2_api.api.id
  name        = "prod"
  auto_deploy = true
}

output "api_url" {
  value = "${aws_apigatewayv2_api.api.api_endpoint}/hello"
}
```

---

### ✅ Deploying:

```bash
terraform init
terraform apply
```

You’ll get a **public HTTP endpoint** like:

```
https://abc123.execute-api.us-east-1.amazonaws.com/hello
```

