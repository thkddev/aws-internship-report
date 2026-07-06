---
title: "Configure Backend & Test APIs"
date: 2026-07-03
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

After the infrastructure is deployed, this step walks through creating the first user, uploading a document, and verifying the full end-to-end flow.

#### 1. Create the First Admin User

Since `selfSignUpEnabled: false`, all users must be created by an administrator.

Go to **AWS Console → Cognito → User Pools → `dms-dev` → Users → Create user**:
- **Email**: `admin@yourcompany.com`
- **Temporary password**: Set a strong one (will be changed on first login)
- After creation, go to **Groups** tab and add the user to `SYSTEM_ADMIN`

Or use the AWS CLI:
```bash
aws cognito-idp admin-create-user \
  --user-pool-id <UserPoolId> \
  --username admin@yourcompany.com \
  --temporary-password "Temp@12345!" \
  --user-attributes Name=email,Value=admin@yourcompany.com Name=email_verified,Value=true Name=name,Value="Admin User"

aws cognito-idp admin-add-user-to-group \
  --user-pool-id <UserPoolId> \
  --username admin@yourcompany.com \
  --group-name SYSTEM_ADMIN
```

#### 2. Authenticate and Get JWT Token

Use the Cognito `USER_PASSWORD_AUTH` flow to get an access token:

```bash
aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id <UserPoolClientId> \
  --auth-parameters USERNAME=admin@yourcompany.com,PASSWORD="YourNewPassword"
```

Copy the `AccessToken` from the response. You will use this as a Bearer token in all API calls.

#### 3. Test the `/me` Endpoint

```bash
curl -H "Authorization: Bearer <AccessToken>" \
  https://<ApiUrl>/me
```

Expected response:
```json
{
  "userId": "...",
  "email": "admin@yourcompany.com",
  "name": "Admin User",
  "groups": ["SYSTEM_ADMIN"]
}
```

![curl user](../../images/5.4curluser.png)

#### 4. Upload a Document (Full Flow)

**Step 4a — Request a Presigned Upload URL:**
```bash
curl -X POST \
  -H "Authorization: Bearer <AccessToken>" \
  -H "Content-Type: application/json" \
  -d '{"filename": "report.pdf", "contentType": "application/pdf", "size": 102400}' \
  https://<ApiUrl>/upload-intents
```

Response includes:
```json
{
  "uploadIntentId": "ui-xxxx",
  "presignedUrl": "https://s3.amazonaws.com/quarantine-bucket/quarantine/...",
  "expiresAt": "2026-07-03T14:00:00Z"
}
```

**Step 4b — Upload directly to S3 using the Presigned URL:**
```bash
curl -X PUT \
  -H "Content-Type: application/pdf" \
  --data-binary @report.pdf \
  "<presignedUrl>"
```

![Upload to S3](../../images/5.4anhluufilevaos3.png)

**Step 4c — Verify the pipeline ran automatically:**
1. Go to **S3 → QuarantineBucket** → verify the file appeared
2. Go to **GuardDuty → Malware Protection** → verify the scan ran and tagged the file
3. Go to **S3 → DocumentsBucket** → the clean file should appear here after ~30 seconds
   ![File saved](../../images/5.4fileluu.png)
4. Go to **DynamoDB → `dms-dev`** → check for a new `DOC#...` record

#### 5. List Documents

```bash
curl -H "Authorization: Bearer <AccessToken>" \
  https://<ApiUrl>/documents
```

You should see the uploaded document in the response with its `documentId`, `filename`, `versionNumber`, and timestamps.

![curl document](../../images/5.4curldocument.png)

#### 6. Run the Frontend Locally

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173` in your browser. Enter the `UserPoolId`, `UserPoolClientId`, and `ApiUrl` from the CDK outputs to configure the app, then log in with your admin credentials.

![Document List](../../images/5.4danhsachdoc.png)

![Department Document](../../images/5.4phongbankinhdoanhfile.png)
