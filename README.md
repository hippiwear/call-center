# call-center
# Hippolyte Architect of this project. 
# Date: 01/03/2025


# Call Center Setup Using Amazon Lex, Amazon Connect, and AWS Lambda

## 1. Introduction

### 1.1 Purpose
This document outlines the configuration and implementation of a call center using Amazon Lex, Amazon Connect, and AWS Lambda. The goal is to streamline customer interactions by routing calls effectively based on their intent.

### 1.2 Overview of the Call Center Setup
The call center leverages Amazon Lex for conversational AI, Amazon Connect for call routing, and AWS Lambda for custom logic. It includes multiple queues for handling orders, inquiries, and complaints.

---

## 2. Amazon Lex Configuration

### 2.1 Intents and Slots
Amazon Lex is configured with specific intents to identify customer needs. Slots capture additional details like order ID or customer name.

### 2.2 Utterances for Intents
Sample utterances for each intent are provided to guide users through interactions.

### 2.3 Prompt Configurations
Custom prompts ensure a natural flow for user interactions.

---

## 3. Amazon Connect Configuration

### 3.1 Setting Up Call Flows
Amazon Connect call flows define the customer journey from the initial call to resolution.

### 3.2 Queues Configuration
- **Order Queue**: Invoke lambda function.
- **Inquiry Queue**: Manages general questions.
- **Complaint Queue**: Resolves customer complaints.

### 3.3 Agent Profiles
Agent profiles ensure calls are routed to the appropriate personnel:
- **John Perry**: Inquiry handler.
- **Douglas Smith**:git  complaint handler.

---

## 4. Lambda Function Integration

### 4.1 Purpose of Lambda Function
AWS Lambda enables dynamic logic for order processing and other functionalities.

### 4.2 Lambda Function: Retrieving Order Information

**Lambda Function Code**
Below is the Python code for the AWS Lambda function that retrieves order information from DynamoDB and brings it back to the contact flow:

```python
import boto3
import json

# Initialize DynamoDB client
dynamodb = boto3.client('dynamodb')

def lambda_handler(event, context):
    try:
        # Retrieve order ID from the input event
        order_id = event['Details']['Parameters']['orderId']
        
        # Get item from DynamoDB
        response = dynamodb.get_item(
            TableName='OrdersTable',  # Replace with your DynamoDB table name
            Key={
                'orderId': {'S': order_id}
            }
        )
        
        # Check if item exists
        if 'Item' in response:
            order_info = response['Item']
            # Create a response for Amazon Connect
            return {
                'statusCode': 200,
                'body': json.dumps({
                    'orderStatus': order_info['status']['S'],
                    'deliveryDate': order_info['deliveryDate']['S']
                })
            }
        else:
            return {
                'statusCode': 404,
                'body': 'Order not found'
            }
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': f"Internal server error: {str(e)}"
        }
```

### 4.3 Setting Up Permissions for Lambda in Amazon Connect

To allow your Lambda function to be called from Amazon Connect:
1. Open the **Amazon Connect Console**.
2. Navigate to **Contact Flows** > **AWS Lambda**.
3. Add the Lambda function you created earlier to the list of allowed functions.

### 4.4 Configuring DynamoDB Permissions for Lambda

To allow your Lambda function to retrieve data from DynamoDB:
1. Attach the following policy to the IAM role associated with your Lambda function:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "dynamodb:GetItem",
            "Resource": "arn:aws:dynamodb:<region>:<account-id>:table/OrdersTable"
        }
    ]
}
```

Replace `<region>`, `<account-id>`, and `OrdersTable` with your specific AWS region, account ID, and table name.

### 4.5 Testing the Lambda Function

To test the Lambda function:
1. Navigate to the **Lambda Console**.
2. Select your function.
3. Use the following test event to simulate input from Amazon Connect:

```json
{
    "Details": {
        "Parameters": {
            "orderId": "12345"
        }
    }
}
```

4. Verify the output matches the expected response format.

---

## 5. Flow Visualization

### 5.1 Overall Flow Diagram
A high-level diagram illustrates the end-to-end flow of the call center. (see image infovis im.png in projectfiles)

### 5.2 Detailed Steps with Blocks
Each block in the flow is explained, highlighting key actions.

### 5.3 Conditions and Routing
Conditions for routing decisions are defined based on customer input.

---

## 6. CloudWatch Monitoring

### 6.1 Integration with Amazon Connect, Lex, and Lambda
Logs from all services are integrated into CloudWatch for monitoring and debugging.

### 6.2 Exploring CloudWatch Logs
- **Amazon Connect Logs**: Track call flow events.
- **Amazon Lex Logs**: Monitor conversation interactions.
- **Lambda Logs**: Debug execution logic.

---

## 7. Testing and Deployment

### 7.1 Testing Call Flows
Call flows are tested to ensure seamless routing and accurate responses.

### 7.2 Validating Agent Routing
Agent-specific routing is verified to match customer needs.

### 7.3 Lambda Function Validation
Functional testing ensures Lambda integration works as expected.

---

## 8. Final Flow Visualization

### 8.1 Visual Representation of the Flow
The final flow diagram consolidates all elements of the call center setup. (see image infovis im.png in projectfiles)

### 8.2 Key Highlights
Highlights of the implementation include efficient routing and robust integration.

---



