# s3-to-s3-automation


## 🧭 Goal:

* Upload to **`bucket1`** → triggers CodePipeline → deploys to **`bucket2`** (static website)

---

## ✅ Step 1: Enable Event Notifications in Source Bucket (`bucket1`)

### 🔹 In S3 Console:

1. Go to **S3 > bucket1**
2. Go to **Properties** tab
3. Scroll to **Event notifications** > Click **“Create event notification”**
4. **Name**: `TriggerPipelineOnUpload`
5. **Event types**: Check **"All object create events"**
6. **Prefix**: Leave blank or specify if needed
7. **Send to**: Choose **EventBridge**

📌 This allows S3 to emit events to EventBridge when a file is uploaded.

---

## ✅ Step 2: Create Target Bucket (`bucket2`) for Static Hosting

### 🔹 In S3 Console:

1. Go to **S3 > Create bucket**
2. Name: `bucket2`
3. Disable **“Block all public access”**
4. After creation:

   * Go to **Properties** tab
   * Enable **Static website hosting**

     * Index document: `index.html`
     * (Optional) Error document: `error.html`
   * Note the **bucket website endpoint** (e.g., `http://bucket2.s3-website-us-east-1.amazonaws.com`)

---

## ✅ Step 3: Create CodePipeline

### 🔹 In CodePipeline Console:

1. **Create pipeline**
2. **Pipeline name**: `your-pipeline`
3. **Service Role**:

   * Choose **"New service role"** or your own existing IAM role
4. **Click Next**

### 🧩 Source Stage:

1. Source provider: **Amazon S3**
2. Bucket: `bucket1`
3. **S3 object key**: your-zip-file
4. Output artifact name: `SourceOutput`

### 🛠️ Build Stage: skip
### 📦 Deploy Stage: skip

1. **Deploy provider**: **Amazon S3**
2. Bucket: `bucket2`
3. Leave deployment path blank so unzipped will happen in root
4. Extract file before deploy: ✅ Yes (to unpack zipped HTMLs, etc.)
5. Input artifact: `SourceOutput`

> 💡 If uploading zipped content (e.g., `site.zip`), CodePipeline will unzip it before deploying to `bucket2`.

---

## ✅ Step 4: Set Permissions for Static Website (Public Read)

### 🔹 In `bucket2` > Permissions > Bucket policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::bucket2/*"
        }
    ]
}
```
