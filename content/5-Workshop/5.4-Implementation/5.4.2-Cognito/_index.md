---
title : "Auth Layer — Cognito"
date : 2026-07-14
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

#### Goal

Set up Amazon Cognito to handle user sign-up, login, and JWT token issuance through a Hosted UI, so the API never has to handle passwords directly.

#### Step 1 — Start Creating a User Pool

1. Open the **Cognito** console.
![picture](/images/5-Workshop/5.4-S3-onprem/cognito.jpeg)
2. Click **Create user pool**.
![picture](/images/5-Workshop/5.4-S3-onprem/create_user_pool.jpeg)
3. On the **Configure options** screen, under **Options for sign-in identifiers**, check **Email**. Leave **Phone number** and **Username** unchecked.
![picture](/images/5-Workshop/5.4-S3-onprem/checked_email.jpeg)
4. Under **Self-registration**, leave **Enable self-registration** checked.
![picture](/images/5-Workshop/5.4-S3-onprem/self.jpeg)
5. Under **Add a return URL**, enter:
![picture](/images/5-Workshop/5.4-S3-onprem/url.jpeg)
6. Scroll down and choose any development platform option .
![picture](/images/5-Workshop/5.4-S3-onprem/python_platform.jpeg)
7. Click **Create user directory**.
![picture](/images/5-Workshop/5.4-S3-onprem/click_create.jpeg)

AWS creates the pool immediately with an auto-generated name (e.g. User pool - nwdtvm). This is expected — it's renamed in the next step.

#### Step 2 — Rename the User Pool and App Client

1. From the success screen, click the **Amazon Cognito** breadcrumb to return to the pool list.
![picture](/images/5-Workshop/5.4-S3-onprem/cognito.jpeg)
2. Click into the new pool.
![picture](/images/5-Workshop/5.4-S3-onprem/user_pool.jpeg)
3. On the **User pool overview** tab, click the pencil icon next to the pool name. Rename to:
![picture](/images/5-Workshop/5.4-S3-onprem/remane_pool.jpeg)
![picture](/images/5-Workshop/5.4-S3-onprem/name_pool.jpeg)
4. Go to the **App integration** tab → **App clients and analytics** 
![picture](/images/5-Workshop/5.4-S3-onprem/app_client.jpeg)
5. Click into the created app client.
![picture](/images/5-Workshop/5.4-S3-onprem/Create_client.jpeg)
5. Click **Edit**, rename to:
![picture](/images/5-Workshop/5.4-S3-onprem/edit_client.jpeg)
![picture](/images/5-Workshop/5.4-S3-onprem/client_name.jpeg)

#### Step 3 — Verify Sign-In and Sign-Up Defaults

1. **Sign-in experience** tab → confirm password policy requires a minimum of **8 characters**.
2. **Sign-up experience** tab → confirm email verification is enabled.
3. **Messaging** tab → default Cognito-managed email is used, no changes needed.

#### Step 4 — Set Up the Hosted UI Domain

1. In the left sidebar, click **Branding** → **Domain**.
2. Click **Actions** → **Create Cognito domain** (or edit an existing placeholder).
3. Enter the domain prefix:
   ```
   pathbridger
   ```
4. Click **Save changes**.

This produces the full Hosted UI domain:
```
pathbridger.auth.ap-southeast-1.amazoncognito.com
```

#### Step 5 — Configure the App Client's OAuth Settings

1. **App integration** tab → click into doc-summarizer-app-client.
2. Find **Hosted UI** / **Login pages configuration** → **Edit**.
3. Under **Allowed callback URLs**, confirm http://localhost:8000 is present.
4. Under **OAuth 2.0 grant types**, confirm **Authorization code grant** is selected.
5. Under **OpenID Connect scopes**, confirm **Email**, **OpenID**, and **Profile** are all selected.
6. Under **Identity providers**, confirm **Cognito user pool** is selected.
7. Click **Save changes**.

#### Step 6 — Enable USER_PASSWORD_AUTH for Automated Testing

The Hosted UI flow (Steps 1–5) is used by real users signing in through a browser. Automated load tests (Section 5.6) can't drive a browser redirect, so a second auth flow is needed for scripted access.

1. Still inside doc-summarizer-app-client, find the **Authentication flows** section → **Edit**.
2. Check **ALLOW_USER_PASSWORD_AUTH** and **ALLOW_REFRESH_TOKEN_AUTH**.
3. Click **Save changes**.

This enables the InitiateAuth API (called directly via boto3, unauthenticated/unsigned), which is distinct from the Hosted UI's /oauth2/token endpoint — that endpoint only accepts authorization_code/refresh_token grants and rejects grant_type=password.

#### Step 7 — Note Your Cognito Values

1. **User pool overview** tab → note the **User pool ID**.
2. **App clients** tab → click the app client → note the **Client ID**.
3. **App integration** tab → **Domain** section → note the **Cognito domain**.

| Value | Actual value |
|---|---|
| User pool ID | ap-southeast-1_Uo593E4hR |
| Client ID | 7mfke3ntkous3rbvpbqpu7c2nb |
| Cognito domain | pathbridger.auth.ap-southeast-1.amazoncognito.com |

**How to verify:** Click **View login page** from the pool overview — this opens the live Hosted UI sign-in screen for doc-summarizer-app-client. A working page here confirms the domain and app client are correctly linked.

#### How JWT Authentication Works

1. Browser redirects to the Hosted UI at pathbridger.auth.ap-southeast-1.amazoncognito.com.
2. User signs in with email and password.
3. Cognito verifies credentials.
4. Cognito redirects back with an authorization code.
5. The app exchanges the code for a JWT (id_token) at the /oauth2/token endpoint.
6. The JWT is sent as the Authorization header on every API call.
7. API Gateway's Cognito authorizer validates the token before invoking Lambda.

The JWT's sub claim becomes the user_id used in DynamoDB (see Section 4.1), ensuring each user only sees their own history.

#### Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| Red warning "you must select at least one sign-in identifier" | No sign-in option checked on the Configure options screen | Check **Email** before continuing |
| invalid_request on the Hosted UI login page | Callback URL in the app client doesn't match what the app is using | Confirm **Allowed callback URLs** exactly matches http://localhost:8000 |
| grant_type=password returns 400 from /oauth2/token | The Hosted UI's OAuth token endpoint only supports authorization_code/refresh_token | Use InitiateAuth with USER_PASSWORD_AUTH instead (Step 6) — this is a separate, unauthenticated Cognito Identity Provider API, not the Hosted UI endpoint |
| Domain step shows no "Create Cognito domain" option | Already viewing App Integration inside the app client rather than the user pool level | Navigate to **Branding → Domain** at the user pool level, not inside the app client's login pages config |