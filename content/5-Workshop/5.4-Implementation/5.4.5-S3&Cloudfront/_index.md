---
title : "Frontend Hosting — S3 & CloudFront"
date : 2026-07-14
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

#### Goal

Host the static frontend (frontend/index.html, frontend/script.js) on S3, serve it securely through CloudFront, and update Cognito and CORS settings to trust the new domain. This module is not yet deployed at the time of writing — the steps below are what needs to happen, not a record of what's already live.

#### Step 1 — Create the S3 Bucket

1. Open the **S3** console → **Create bucket**.
2. Bucket name: doc-summarizer-frontend-YOUR_ACCOUNT_ID (S3 bucket names must be globally unique — the account ID suffix avoids collisions with other workshop readers).
3. Region: ap-southeast-1.
4. Under **Block Public Access settings**, leave all four boxes checked (block all public access). The bucket will be reached only through CloudFront, never directly.
5. Click **Create bucket**.
6. Upload frontend/index.html and frontend/script.js (and any other static assets) to the bucket root.

#### Step 2 — Create a CloudFront Distribution with Origin Access Control

1. Open the **CloudFront** console → **Create distribution**.
2. **Origin domain**: select the S3 bucket created in Step 1.
3. **Origin access**: select **Origin access control settings (recommended)** → **Create new OAC** → accept the defaults → **Create**.
4. **Viewer protocol policy**: **Redirect HTTP to HTTPS**.
5. **Default root object**: index.html.
6. Click **Create distribution**. Note the **Distribution domain name** — this becomes the frontend's public URL, e.g. [FILL IN: distribution domain name, e.g. dxxxxxxxxxxxxx.cloudfront.net].

#### Step 3 — Update the S3 Bucket Policy

CloudFront's OAC needs explicit permission to read from the bucket. After creating the distribution, CloudFront shows a banner offering to update the bucket policy automatically.

1. Return to the CloudFront distribution → click the banner **Copy policy** (or **Go to S3 bucket permissions**).
2. If not applied automatically, open the S3 bucket → **Permissions** tab → **Bucket policy** → **Edit** → paste the policy CloudFront generated, replacing YOUR_DISTRIBUTION_ID and YOUR_BUCKET_NAME with actual values:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

3. Click **Save changes**.

**How to verify:** Open the CloudFront distribution domain name in a browser. The frontend loads. A direct request to the S3 bucket's object URL (bypassing CloudFront) returns AccessDenied.

#### Step 4 — Wire script.js to the Real API URL

1. Open frontend/script.js.
2. Set the API base URL to the API Gateway invoke URL from Section 4.3:
   ```js
   const API_BASE_URL = "https://kqtcvcv5g0.execute-api.ap-southeast-1.amazonaws.com/v1";
   ```
3. Set the API key (from Section 4.3, Step 5) and Cognito values (from Section 4.2, Step 7) as needed by the frontend's auth handling.
4. Re-upload the updated script.js to the S3 bucket (Step 1).
5. Create a CloudFront invalidation so the new file is served immediately instead of a cached old version:
   - CloudFront console → distribution → **Invalidations** tab → **Create invalidation** → path /script.js (or /* to invalidate everything) → **Create invalidation**.

#### Step 5 — Update Cognito Callback URLs

The Cognito app client currently only trusts http://localhost:8000 as a callback URL (Section 4.2). Once the frontend has a real domain, that domain needs to be added so sign-in redirects work from the live site.

1. Cognito console → doc-summarizer-user-pool → **App integration** → doc-summarizer-app-client → **Hosted UI** → **Edit**.
2. Under **Allowed callback URLs**, click **Add another URL** and add:
   ```
   https://[FILL IN: your CloudFront distribution domain name]
   ```
3. Click **Save changes**.
4. Keep the localhost entry alongside it — useful for continued local development.

#### Step 6 — Lock CORS to the Real Domain

API Gateway's CORS configuration was set to Access-Control-Allow-Origin: '*' during initial development (Section 4.3). This is a reasonable default while there's no real frontend domain yet, but it should be tightened once one exists — a wildcard origin means any website can call this API from a user's browser.

1. API Gateway console → doc-summarizer-api → select the /summarize resource → **CORS** → **Edit**.
2. Change **Access-Control-Allow-Origin** from * to:
   ```
   https://[FILL IN: your CloudFront distribution domain name]
   ```
3. Repeat for /history.
4. Click **Deploy API** → stage v1 to apply the change.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| AccessDenied when loading the CloudFront URL | Bucket policy wasn't updated to trust the distribution's OAC | Repeat Step 3, confirm the distribution ID and bucket name in the policy are correct |
| Frontend loads but API calls fail with a CORS error in the browser console | CORS origin locked to the CloudFront domain, but the request is coming from localhost during testing, or vice versa | Keep both origins allowed during development, or test consistently from the same origin the CORS policy trusts |
| Old script.js still being served after re-upload | CloudFront caches objects at edge locations | Create a CloudFront invalidation (Step 4) after every frontend file update |
| Cognito Hosted UI redirects to a blank/error page after sign-in from the live site | CloudFront domain not yet added to Cognito's allowed callback URLs | Complete Step 5 |