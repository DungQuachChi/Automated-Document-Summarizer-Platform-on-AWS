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
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/create_api.jpeg)

2. Select **REST API** (not HTTP API) → **Build**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/build_rest.jpeg)

3. API name: doc-summarizer-api.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/api_detail.jpeg)

4. Endpoint type: **Regional**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/endpoint_type.jpeg)

5. Click **Create API**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/click_create.jpeg)

#### Step 2 — Create a Cognito Authorizer

1. In the left sidebar, click **Authorizers** → **Create authorizer**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/authorizer.jpeg)
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/create_auth.jpeg)

2. Fill in:
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/auth_detaildetail.jpeg)

3. Click **Create authorizer**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/click_auth.jpeg)

#### Step 3 — Create the /summarize Route

**Create the resource:**

1. Click **Resources** → **Create resource**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/Resource_create.jpeg)

2. Resource name: summarize and check **CORS**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/check_Resources.jpeg)

3. Click **Create resource**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/click_auth.jpeg)


**Create the POST method:**

1. Select /summarize → **Create method**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/method_create.jpeg)

2. Method type: **POST**.Integration type: **Lambda function**.Check **Lambda proxy integration**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/Method_type.jpeg)

3. Function: doc-summarizer-fn.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/method_Function.jpeg)

6. Click **Create method**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/method_click.jpeg)


**Attach the Cognito authorizer:**

1. Click the **POST** method → **Method request** → **Edit**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/sumarize_method/post.jpeg)

2. Authorization: select cognito-authorizer.**API key required**, select **true**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/sumarize_method/edit.jpeg)

3. Click **Save**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/sumarize_method/save.jpeg)


#### Step 4 — Create the /history Route

**Create the resource:**

1. Click **Create resource** (at root /).
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/Create_method.jpeg)

2. Resource name: history.Check **CORS**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/edit.jpeg)

3. Click **Create resource**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/clickCreate.jpeg)


**Create the GET method:**

1. Select /history → **Create method**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/create_get.jpeg)

2. Method type: **GET**.Integration type: **Lambda function**.Check **Lambda proxy integration**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/edit_type.jpeg)

3. Lambda function: doc-summarizer-fn.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/provide.jpeg)

4. Click **Create method**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/clickCreate.jpeg)




**Attach the Cognito authorizer:**

1. Click the **GET** method → **Method request** → **Edit**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/history_method/click_get.jpeg)

2. Authorization: select cognito-authorizer. Checked **API key required**
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/sumarize_method/edit.jpeg)

3. Click **Save**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/sumarize_method/save.jpeg)

#### Step 5 — Create a Usage Plan and API Key

The usage plan limits request rate and monthly volume — this is what stands between the API and an unexpectedly large AWS bill if a client misbehaves or a key leaks.

1. In the left sidebar, click **Usage Plans** → **Create**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/usagePlan.jpeg)

2. Fill in:Name: doc-summarizer-usage-plan. 
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/nameUsage.jpeg)

3. Throttling: rate 2 requests/second. Quota: 5000 requests / month
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/rateandreq.jpeg)

4. Click **Create usage plan**. 
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/createplan.jpeg)

#### Step 6 — Deploy the API

1. Click **Deploy API**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/Deploy API.jpeg)

2. Stage: **New stage**, name: v1. Click **Deploy**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/DeployAPI_detail.jpeg)

3. Copy the **Invoke URL** — this is the base URL for all requests
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/copy_invoke.jpeg)

#### Step 7 — Add API key
4. Click **Associated API key** → **Add API key**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/add_api.jpeg)

5. Create a new API key, name it doc-summarizer-api-key.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/APIKEY_detail.jpeg)

6. Click **Done**.
![picture](/images/5-Workshop/5.4-Implementation/5.4.3-API/UsagePLan/click_add_api.jpeg)


#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| {"message": "Missing Authentication Token"} | Request sent to a path/method that doesn't exist on the deployed stage | Confirm the URL includes the stage name (/v1/summarize, not just /summarize) |
| {"message": "Unauthorized"} | No Authorization header, or an expired/invalid JWT | Sign in again via the Hosted UI to get a fresh token (Section 4.2) |
| {"message": "Forbidden"} | Missing or invalid x-api-key header | Confirm the request includes x-api-key: <your API key value> |
| Changes made in the console don't appear when calling the API | API Gateway requires a redeploy after any resource/method/authorizer change | Repeat Step 6 — click **Deploy API** and select the existing v1 stage |
| Usage plan shows 0 associated stages | Stage didn't exist yet when the usage plan was created | Add the stage association after deploying — see the note at the end of Step 6 |