---
title : "Backend Foundation — DynamoDB & Lambda"
date : 2026-07-14
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

#### Goal

Create the storage layer (DynamoDB table with a GSI for date-based queries) and the compute layer (a Lambda function with a least-privilege IAM role) that everything else in this workshop will connect to.

#### Step 1 — Create the DynamoDB Table

1. Open the **DynamoDB** console.
2. Click **Create table**.
3. Fill in:

   | Field | Value |
   |---|---|
   | Table name | SummarizerTable |
   | Partition key | user_id (String) |
   | Sort key | timestamp (String) |

4. Under **Table settings**, select **Customize settings**.
5. Under **Read/write capacity settings**, select **On-demand**. This bills per request instead of provisioned throughput — no cost when the table is idle.
6. Under **Encryption at rest**, leave the default (AWS owned key) — this satisfies SSE.
7. Expand **Additional settings** and enable **Point-in-time recovery (PITR)**. PITR lets you restore the table to any point in the last 35 days if data is accidentally deleted or corrupted.
8. Click **Create table**. Wait for status to change to **Active**.

#### Step 2 — Add the GSI

The Global Secondary Index (summary-date-index) lets you query all summaries created on a specific date, independent of which user made them — used later by the weekly report Lambda.

1. Open the SummarizerTable table → click the **Indexes** tab.
2. Click **Create index**.
3. Fill in:

   | Field | Value |
   |---|---|
   | Index name | summary-date-index |
   | Partition key | summary_date (String) |
   | Sort key | timestamp (String) |

4. Read/write capacity: **On-demand** (inherits from the table).
5. Click **Create index**. Wait for status **Active** — this can take a few minutes on an existing table.

**How to verify:** In the **Indexes** tab, summary-date-index shows status **Active** with partition key summary_date.

#### Step 3 — Create the IAM Role for Lambda

1. Open the **IAM** console → **Roles** → **Create role**.
2. Trusted entity type: **AWS service**. Use case: **Lambda**. Click **Next**.
3. Skip attaching any AWS-managed policy — click **Next** without checking anything.
4. Role name: doc-summarizer-lambda-role. Click **Create role**.
5. Click into the new role → **Add permissions** → **Create inline policy**.
6. Switch to the **JSON** tab and paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "logs:CreateLogGroup",
      "Resource": "arn:aws:logs:ap-southeast-1:YOUR_ACCOUNT_ID:*"
    },
    {
      "Effect": "Allow",
      "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:ap-southeast-1:YOUR_ACCOUNT_ID:log-group:/aws/lambda/doc-summarizer-fn:*"
    },
    {
      "Effect": "Allow",
      "Action": ["dynamodb:PutItem", "dynamodb:Query"],
      "Resource": [
        "arn:aws:dynamodb:ap-southeast-1:YOUR_ACCOUNT_ID:table/SummarizerTable",
        "arn:aws:dynamodb:ap-southeast-1:YOUR_ACCOUNT_ID:table/SummarizerTable/index/summary-date-index"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "bedrock:InvokeModel",
      "Resource": "arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-lite-v1:0"
    },
    {
      "Effect": "Allow",
      "Action": "cloudwatch:PutMetricData",
      "Resource": "*",
      "Condition": {
        "StringEquals": { "cloudwatch:namespace": "Custom/Bedrock" }
      }
    }
  ]
}
```

Replace YOUR_ACCOUNT_ID with your account ID (from aws sts get-caller-identity). This is a least-privilege policy — no dynamodb: or bedrock: wildcard, no access to any table or model besides the ones this project uses, and the CloudWatch permission is scoped to a single metric namespace.

7. Click **Next**, name the policy doc-summarizer-lambda-policy, click **Create policy**.

#### Step 4 — Create the Lambda Function

1. Open the **Lambda** console → **Create function**.
2. Fill in:

   | Field | Value |
   |---|---|
   | Function name | doc-summarizer-fn |
   | Runtime | Python 3.12 |
   | Architecture | x86_64 |

3. Under **Permissions**, select **Use an existing role** → doc-summarizer-lambda-role.
4. Click **Create function**.

#### Step 5 — Configure Function Settings

1. **Configuration** tab → **General configuration** → **Edit**.
2. Set **Timeout** to 30 sec (Bedrock calls can take several seconds, and this workshop's Lambda includes retry logic).
3. Set **Memory** to 256 MB.
4. Click **Save**.

#### Step 6 — Set Environment Variables

1. **Configuration** tab → **Environment variables** → **Edit** → **Add environment variable**.
2. Add each of the following:

   | Key | Value |
   |---|---|
   | DYNAMODB_TABLE | SummarizerTable |
   | BEDROCK_MODEL_ID | us.amazon.nova-lite-v1:0 |

3. Click **Save**.

Note: MOCK_SUMMARIZE is a separate flag used by the local FastAPI dev app and frontend, not by this Lambda function — this Lambda always calls the real Bedrock endpoint. Section 4.4 covers requesting Bedrock model access, and what happens if you hit the daily quota before that access is fully provisioned.

#### Step 7 — Write and Deploy the Lambda Code

1. **Code** tab → delete the default code in lambda_function.py.
2. Paste the following:

```python
import json
import boto3
import os
import time
from datetime import datetime, timezone
from botocore.exceptions import ClientError, ReadTimeoutError, ConnectTimeoutError

class DailyQuotaExceededError(Exception):
    pass

dynamodb = boto3.resource('dynamodb')
# ap-southeast-1 is not in the AP Nova inference pool; route cross-region to us-east-1
bedrock_client = boto3.client('bedrock-runtime', region_name='us-east-1')
cloudwatch_client = boto3.client('cloudwatch')

TABLE_NAME = os.environ['DYNAMODB_TABLE']
table = dynamodb.Table(TABLE_NAME)

def invoke_bedrock_with_retries(text, max_retries=3):
    model_id = os.environ.get('BEDROCK_MODEL_ID', 'us.amazon.nova-lite-v1:0')
    # Nova uses the Messages API (not Titan's inputText format)
    payload = {
        "messages": [
            {
                "role": "user",
                "content": [{"text": f"Summarize the following text concisely in 2-3 sentences:\n\n{text}"}]
            }
        ],
        "inferenceConfig": {
            "maxTokens": 300,
            "temperature": 0.2,
            "topP": 0.9
        }
    }
    for attempt in range(max_retries):
        try:
            start_time = time.time()
            response = bedrock_client.invoke_model(
                modelId=model_id,
                contentType='application/json',
                accept='application/json',
                body=json.dumps(payload)
            )
            end_time = time.time()
            latency_ms = (end_time - start_time) * 1000
            cloudwatch_client.put_metric_data(
                Namespace='Custom/Bedrock',
                MetricData=[
                    {'MetricName': 'Latency', 'Value': latency_ms, 'Unit': 'Milliseconds'},
                    {'MetricName': 'SuccessCount', 'Value': 1, 'Unit': 'Count'}
                ]
            )
            response_body = json.loads(response['body'].read())
            return response_body['output']['message']['content'][0]['text'].strip()

        except (ClientError, ReadTimeoutError, ConnectTimeoutError) as e:
            error_code = e.response['Error']['Code']
            error_msg = str(e).lower()
            is_daily_quota = (
                error_code == 'ThrottlingException'
                and ('daily' in error_msg or 'per day' in error_msg or 'toomanytokens' in error_msg)
            )
            if is_daily_quota:
                cloudwatch_client.put_metric_data(
                    Namespace='Custom/Bedrock',
                    MetricData=[{
                        'MetricName': 'BedrockErrors',
                        'Value': 1,
                        'Unit': 'Count',
                        'Dimensions': [{'Name': 'ErrorType', 'Value': 'DailyQuotaExceeded'}]
                    }]
                )
                print(f"Bedrock API Error: {str(e)}")
                raise DailyQuotaExceededError(str(e))

            if error_code in ['ThrottlingException', 'ModelTimeoutException'] and attempt < max_retries - 1:
                sleep_time = 2 ** attempt
                time.sleep(sleep_time)
                continue

            cloudwatch_client.put_metric_data(
                Namespace='Custom/Bedrock',
                MetricData=[{
                    'MetricName': 'BedrockErrors',
                    'Value': 1,
                    'Unit': 'Count',
                    'Dimensions': [{'Name': 'ErrorType', 'Value': error_code}]
                }]
            )
            print(f"Bedrock API Error: {str(e)}")
            raise e

def lambda_handler(event, context):
    http_method = event.get('httpMethod')
    path = event.get('path')
    try:
        user_id = event['requestContext']['authorizer']['claims']['sub']
    except KeyError:
        # Dev-only fallback — must be removed/confirmed before any real production deploy
        user_id = "local-test-user-123"

    try:
        if http_method == 'POST' and path == '/summarize':
            return handle_summarize(event, user_id)
        elif http_method == 'GET' and path == '/history':
            return handle_history(user_id)
        else:
            return build_response(404, {"message": "Route Not Found"})
    except Exception as e:
        print(f"Error processing request: {str(e)}")
        return build_response(500, {"message": "Internal server error"})

def handle_summarize(event, user_id):
    body = json.loads(event.get('body', '{}'))
    original_text = body.get('text', '')

    if not original_text:
        return build_response(400, {"message": "Text field is required"})
    if len(original_text) < 20:
        return build_response(400, {"message": "Text must be at least 20 characters"})
    if len(original_text) > 5000:
        return build_response(400, {"message": "Text exceeds maximum length of 5000 characters"})

    try:
        summary = invoke_bedrock_with_retries(original_text)
    except DailyQuotaExceededError:
        return build_response(429, {"message": "Summarization limit reached for today. Please try again after midnight UTC."})
    except Exception:
        return build_response(502, {"message": "AI Summarization service is currently unavailable"})

    now = datetime.now(timezone.utc)
    timestamp = now.isoformat()
    summary_date = now.strftime('%Y-%m-%d')
    item = {
        'user_id': user_id,
        'timestamp': timestamp,
        'summary_date': summary_date,
        'original_text': original_text,
        'summary': summary
    }
    table.put_item(Item=item)
    return build_response(200, item)

def handle_history(user_id):
    response = table.query(
        KeyConditionExpression=boto3.dynamodb.conditions.Key('user_id').eq(user_id),
        ScanIndexForward=False  # newest first
    )
    items = response.get('Items', [])
    return build_response(200, {"history": items})

def build_response(status_code, body):
    return {
        'statusCode': status_code,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps(body)
    }
```

3. Click **Deploy**.

A few design details worth calling out for your report:

- **Cross-region Bedrock call** — the client is hardcoded to us-east-1 because ap-southeast-1 is not in the AP inference pool for Nova Lite. Lambda itself still runs in ap-southeast-1.
- **Exponential backoff retries** — transient errors (ThrottlingException, ModelTimeoutException) retry up to 3 times with 2^attempt second delays. A daily quota error is detected separately and fails fast instead of retrying, since retrying a quota error just burns the timeout window for no benefit.
- **Dev-only auth fallback** — if no Cognito claims are present, user_id falls back to local-test-user-123. This exists for local/manual testing (see Step 8) and must be removed or explicitly confirmed as intentional before any real production deployment.

#### Step 8 — Test with a Mock Event

1. **Test** tab → **Create new event**. Event name: TestSummarize.
2. Paste:

```json
{
  "httpMethod": "POST",
  "path": "/summarize",
  "body": "{\"text\": \"Amazon Web Services offers cloud computing services worldwide. Businesses use AWS to reduce infrastructure costs and scale applications globally.\"}",
  "requestContext": {
    "authorizer": {
      "claims": { "sub": "test-user-123" }
    }
  }
}
```

3. Click **Test**.

**How to verify:** A successful response returns statusCode: 200 with summary, timestamp, and summary_date fields in the body. If Bedrock model access hasn't been granted yet (see Section 4.4), this call will fail with a 502 — that's expected at this point in the workshop.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDeniedException on DynamoDB | IAM policy resource ARN doesn't match table name/region/account ID | Double-check the ARN in Step 3 matches your actual account ID and region exactly |
| ResourceNotFoundException | DYNAMODB_TABLE env var doesn't match the actual table name | Confirm env var value is exactly SummarizerTable (case-sensitive) |
| 502 AI Summarization service is currently unavailable | Bedrock model access not yet granted, or IAM role missing bedrock:InvokeModel | Complete Section 4.4 (Bedrock model access request) |
| 429 Summarization limit reached for today | Daily on-demand quota for the model has been exhausted | Expected behavior — see Section 4.4 for the quota workaround |
| Lambda times out at 3 seconds | Default timeout wasn't changed | Repeat Step 5 — confirm timeout shows 30 sec under General configuration |
| KeyError: 'DYNAMODB_TABLE' | Environment variable wasn't saved | Repeat Step 6, confirm you clicked **Save** after adding the variable |