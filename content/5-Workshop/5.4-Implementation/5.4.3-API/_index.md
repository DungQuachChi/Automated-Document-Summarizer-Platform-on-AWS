---
title : "API Layer — API Gateway"
date : 2026-07-14
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

#### Goal

Set up API Gateway as the secure entry point for the platform: validate JWT tokens via a Cognito authorizer, enforce a usage plan with an API key, and deploy the API to a live stage.

#### Step 1 — Create the REST API

1. Open the **API Gateway** console → **Create API**.
2. Select **REST API** (not HTTP API) → **Build**.
3. API name: doc-summarizer-api.
4. Endpoint type: **Regional**.
5. Click **Create API**.

#### Step 2 — Create a Cognito Authorizer

1. In the left sidebar, click **Authorizers** → **Create authorizer**.
2. Fill in:

   | Field | Value |
   |---|---|
   | Authorizer name | cognito-authorizer |
   | Authorizer type | Cognito |
   | Cognito user pool | doc-summarizer-user-pool |
   | Token source | Authorization |

3. Click **Create authorizer**.

This registers the authorizer but doesn't attach it to any route yet — that happens per-method in Steps 3 and 4.

#### Step 3 — Create the /summarize Route

**Create the resource:**

1. Click **Resources** → **Create resource**.
2. Resource name: summarize.
3. Check **CORS**.
4. Click **Create resource**.

**Create the POST method:**

1. Select /summarize → **Create method**.
2. Method type: **POST**.
3. Integration type: **Lambda function**.
4. Check **Lambda proxy integration**.
5. Lambda function: doc-summarizer-fn.
6. Click **Create method**.

**Attach the Cognito authorizer:**

1. Click the **POST** method → **Method request** → **Edit**.
2. Authorization: select cognito-authorizer.
3. Under **API key required**, select **true**.
4. Click **Save**.

#### Step 4 — Create the /history Route

**Create the resource:**

1. Click **Create resource** (at root /).
2. Resource name: history.
3. Check **CORS**.
4. Click **Create resource**.

**Create the GET method:**

1. Select /history → **Create method**.
2. Method type: **GET**.
3. Integration type: **Lambda function**.
4. Check **Lambda proxy integration**.
5. Lambda function: doc-summarizer-fn.
6. Click **Create method**.

**Attach the Cognito authorizer:**

1. Click the **GET** method → **Method request** → **Edit**.
2. Authorization: select cognito-authorizer.
3. Under **API key required**, select **true**.
4. Click **Save**.

#### Step 5 — Create a Usage Plan and API Key

The usage plan limits request rate and monthly volume — this is what stands between the API and an unexpectedly large AWS bill if a client misbehaves or a key leaks.

1. In the left sidebar, click **Usage Plans** → **Create**.
2. Fill in:

   | Field | Value |
   |---|---|
   | Name | doc-summarizer-usage-plan |
   | Throttling — rate | 2 requests/second |
   | Quota | 5000 requests / month |

3. Click **Next**. Associate the API stage (created in Step 6 — you can return here afterward if the stage doesn't exist yet).
4. Click **Next** → **Add API key to Usage Plan**.
5. Create a new API key, name it doc-summarizer-api-key.
6. Click **Done**.

#### Step 6 — Deploy the API

1. Click **Deploy API**.
2. Stage: **New stage**, name: v1.
3. Click **Deploy**.
4. Copy the **Invoke URL** — this is the base URL for all requests, e.g.:
   ```
   https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1
   ```

If you created the usage plan before the stage existed (Step 5), return to **Usage Plans** → your plan → **API Stages** → **Add API Stage** and associate doc-summarizer-api / v1 now.

#### How to Verify

1. **API Gateway console** → **API Keys** → confirm doc-summarizer-api-key shows an associated usage plan.
2. Retrieve the actual key value: click the key → **Show**.
3. Test that a request without a token and without an API key is rejected — see Section 5.5 for the full test sequence (with token, without token, without key).

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| {"message": "Missing Authentication Token"} | Request sent to a path/method that doesn't exist on the deployed stage | Confirm the URL includes the stage name (/v1/summarize, not just /summarize) |
| {"message": "Unauthorized"} | No Authorization header, or an expired/invalid JWT | Sign in again via the Hosted UI to get a fresh token (Section 4.2) |
| {"message": "Forbidden"} | Missing or invalid x-api-key header | Confirm the request includes x-api-key: <your API key value> |
| Changes made in the console don't appear when calling the API | API Gateway requires a redeploy after any resource/method/authorizer change | Repeat Step 6 — click **Deploy API** and select the existing v1 stage |
| Usage plan shows 0 associated stages | Stage didn't exist yet when the usage plan was created | Add the stage association after deploying — see the note at the end of Step 6 |