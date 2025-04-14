## 📘 **Introduction to DynamoDB (NoSQL)**

### 🔹 What is DynamoDB?

- **NoSQL database** optimized for key-value and document data models.
- **Fully managed**, **serverless**, and scales automatically.
- Provides **single-digit millisecond performance** at any scale.
- Ideal for: real-time apps, IoT, gaming, leaderboards, caching, shopping carts.

---

### 🔍 Key Concepts

| Concept              | Description |
|----------------------|-------------|
| **Table**            | Collection of items (like rows) |
| **Item**             | A single row of data |
| **Attribute**        | A field in an item (like a column) |
| **Primary Key**      | Uniquely identifies each item |
| **Partition Key**    | Required – determines the partition where the item is stored |
| **Sort Key**         | Optional – allows sorting of multiple items with same partition key |
| **Throughput**       | Read/write capacity (On-demand or Provisioned) |
| **Global Secondary Index (GSI)** | Additional indexes for querying |
| **TTL**              | Time-to-live – for automatic item expiry |

---

## 🛠️ **Hands-On: Create & Manage DynamoDB Table**

---

### ✅ Step 1: Create Table via AWS Console

1. Go to [AWS DynamoDB Console](https://console.aws.amazon.com/dynamodb/)
2. Click **"Create table"**
3. Configure:
   - Table name: `Users`
   - Partition key: `UserID` (type: String)
   - No sort key (for now)
4. Capacity mode: **On-demand**
5. Leave other defaults and click **Create Table**

Your DynamoDB table is ready in seconds.

---

### 🧪 Step 2: Add Items (Manually or Programmatically)

Click on your table > **Explore Table Items** > **Create item**:

```json
{
  "UserID": "user123",
  "Name": "Alice",
  "Age": 30,
  "Email": "alice@example.com"
}
```

---

## 🧰 Programmatic Example: Python (Boto3)

```python
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

response = table.put_item(
    Item={
        'UserID': 'user456',
        'Name': 'Bob',
        'Age': 25,
        'Email': 'bob@example.com'
    }
)

print("PutItem succeeded:", response)
```

---

## 🧩 Bonus: Create Table with Terraform

```hcl
resource "aws_dynamodb_table" "users" {
  name           = "Users"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "UserID"

  attribute {
    name = "UserID"
    type = "S"
  }

  tags = {
    Environment = "dev"
    Project     = "NoSQLApp"
  }
}
```

---

## 🔄 Use Cases for DynamoDB

- Serverless web/mobile backends (with AWS Lambda)
- Leaderboards and rankings
- Session stores or token caching
- IoT event data ingestion
- Shopping cart systems


## 🔗 **Use Case: Serverless CRUD API with Lambda + DynamoDB**

### 🧱 What We’re Building

- A **DynamoDB table** for storing users.
- A **Lambda function** for Create/Read/Update/Delete.
- Optional: **API Gateway** to expose this as a REST API.

---

## ✅ Step 1: DynamoDB Table (Terraform or Console)

You can either use this Terraform code:

```hcl
resource "aws_dynamodb_table" "users" {
  name           = "Users"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "UserID"

  attribute {
    name = "UserID"
    type = "S"
  }

  tags = {
    Environment = "dev"
  }
}
```

Or create manually in the console as before.

---

## ✅ Step 2: Lambda Function (Node.js Example)

### 📁 index.js

```js
const AWS = require("aws-sdk");
const dynamo = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
    const { httpMethod, pathParameters, body } = event;

    switch (httpMethod) {
        case "GET":
            const userId = pathParameters?.id;
            const getItem = await dynamo.get({
                TableName: "Users",
                Key: { UserID: userId }
            }).promise();
            return {
                statusCode: 200,
                body: JSON.stringify(getItem.Item)
            };

        case "POST":
            const data = JSON.parse(body);
            await dynamo.put({
                TableName: "Users",
                Item: data
            }).promise();
            return {
                statusCode: 200,
                body: "User created."
            };

        case "DELETE":
            const deleteId = pathParameters?.id;
            await dynamo.delete({
                TableName: "Users",
                Key: { UserID: deleteId }
            }).promise();
            return {
                statusCode: 200,
                body: "User deleted."
            };

        default:
            return {
                statusCode: 400,
                body: "Unsupported method"
            };
    }
};
```

---

## ✅ Step 3: IAM Role for Lambda

Make sure your Lambda execution role has this permission:

```json
{
  "Effect": "Allow",
  "Action": [
    "dynamodb:PutItem",
    "dynamodb:GetItem",
    "dynamodb:DeleteItem"
  ],
  "Resource": "arn:aws:dynamodb:REGION:ACCOUNT_ID:table/Users"
}
```

You can attach this to your Lambda using the AWS Console or Terraform.

---

## ✅ Step 4: Trigger Lambda via API Gateway (Optional)

Use API Gateway to:
- Define routes like `/user/{id}`
- Map HTTP methods (GET, POST, DELETE) to the Lambda
- Handle JSON input/output


## 🧱 What You’ll Get:

- DynamoDB Table: `Users`
- Python Lambda Function: Handles `GET`, `POST`, and `DELETE`
- IAM Role: Grants Lambda access to DynamoDB
- (Optional) API Gateway: REST API to trigger Lambda

---

## ✅ Step 1: Create DynamoDB Table

Either in AWS Console or with Terraform:

```hcl
resource "aws_dynamodb_table" "users" {
  name         = "Users"
  hash_key     = "UserID"
  billing_mode = "PAY_PER_REQUEST"

  attribute {
    name = "UserID"
    type = "S"
  }
}
```

---

## ✅ Step 2: Python Lambda Function

### 📁 `lambda_function.py`

```python
import json
import boto3
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    http_method = event['httpMethod']
    path_params = event.get('pathParameters')
    body = json.loads(event.get('body', '{}'))

    if http_method == 'GET' and path_params:
        user_id = path_params['id']
        response = table.get_item(Key={'UserID': user_id})
        return {
            'statusCode': 200,
            'body': json.dumps(response.get('Item', {}))
        }

    elif http_method == 'POST':
        table.put_item(Item=body)
        return {
            'statusCode': 200,
            'body': 'User added.'
        }

    elif http_method == 'DELETE' and path_params:
        user_id = path_params['id']
        table.delete_item(Key={'UserID': user_id})
        return {
            'statusCode': 200,
            'body': 'User deleted.'
        }

    else:
        return {
            'statusCode': 400,
            'body': 'Unsupported operation.'
        }
```

---

## ✅ Step 3: IAM Role for Lambda

Make sure Lambda has permission to interact with DynamoDB:

```json
{
  "Effect": "Allow",
  "Action": [
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:DeleteItem"
  ],
  "Resource": "arn:aws:dynamodb:<region>:<account_id>:table/Users"
}
```

Attach this to your Lambda's IAM role.

---

## ✅ Step 4: Connect via API Gateway

Create an API Gateway REST API with these mappings:

| Method | Resource       | Lambda Handler             |
|--------|----------------|----------------------------|
| GET    | /user/{id}     | `lambda_handler` (GET)     |
| POST   | /user          | `lambda_handler` (POST)    |
| DELETE | /user/{id}     | `lambda_handler` (DELETE)  |

Make sure to enable **Lambda Proxy Integration** for request/response mapping to work automatically.

---

## 🧪 Example Test Payloads (via Postman or curl)

- **POST /user**

```json
{
  "UserID": "u101",
  "Name": "Alice",
  "Email": "alice@example.com"
}
```

- **GET /user/u101**

Returns Alice’s data.

- **DELETE /user/u101**

Deletes Alice.
