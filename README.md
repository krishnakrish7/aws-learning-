# ☁️ AWS Visual Guides

A collection of interactive, mobile-friendly HTML visual diagrams covering core AWS services and architecture patterns. Hosted on AWS S3 + CloudFront and accessible from any device.

---

## 📂 Files Included

| File | Topic |
|---|---|
| `index.html` | 🏠 Homepage — navigation hub for all guides |
| `aws_vpc_deep_dive.html` | 🌐 VPC Deep Dive |
| `aws_security_layers.html` | 🔐 Security Layers |
| `aws_databases_deep_dive.html` | 🗄️ Databases Deep Dive |
| `aws_storage_services.html` | 💾 Storage Services |
| `aws_ha_loadbalancing_autoscaling.html` | ⚖️ HA, Load Balancing & Auto Scaling |
| `aws_route53_cloudfront_global.html` | 🌍 Route 53, CloudFront & Global Architecture |

---

## 🚀 Hosting on AWS (S3 + CloudFront)

### Prerequisites
- An active AWS account
- All HTML files from this repo

---

### Step 1 — Create an S3 Bucket

1. Go to [AWS Console](https://console.aws.amazon.com) → search **S3**
2. Click **"Create bucket"**
3. Enter a unique bucket name (e.g. `my-aws-diagrams-2024`)
4. Choose your preferred AWS Region
5. Under **"Block Public Access"** → uncheck **"Block all public access"** → confirm the warning
6. Leave everything else as default → click **"Create bucket"**

---

### Step 2 — Upload HTML Files

1. Click on your newly created bucket
2. Click **"Upload"** → **"Add files"**
3. Select all HTML files from this repo (including `index.html`)
4. Click **"Upload"**

---

### Step 3 — Enable Static Website Hosting

1. Inside your bucket → go to **"Properties"** tab
2. Scroll to **"Static website hosting"** → click **"Edit"**
3. Select **"Enable"**
4. Set **Index document** to `index.html`
5. Click **"Save changes"**

---

### Step 4 — Set Bucket Policy (Make Public)

1. Go to **"Permissions"** tab → **"Bucket policy"** → click **"Edit"**
2. Paste the following (replace `YOUR-BUCKET-NAME` with your actual bucket name):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

3. Click **"Save changes"**

---

### Step 5 — Set Up CloudFront (CDN + HTTPS)

1. Go to AWS Console → search **CloudFront** → click **"Create distribution"**

**Origin section:**
- **Origin domain** → select your S3 bucket from the dropdown
- If a yellow popup appears saying *"Use website endpoint"* → click it
- **Origin access** → select **"Public"**

**Default cache behavior section:**
- **Viewer protocol policy** → select **"Redirect HTTP to HTTPS"**
- **Allowed HTTP methods** → leave as **GET, HEAD**

**WAF section:**
- Select **"Do not enable security protections"**

**Settings section:**
- **Price class** → select **"Use only North America and Europe"** (cheapest)
- **Default root object** → type `index.html`

2. Click **"Create distribution"**
3. Wait **5–10 minutes** for status to change from *"Deploying"* to *"Enabled"*
4. Copy the **Distribution domain name** (e.g. `d1xxxxx.cloudfront.net`)
5. Open it on your phone — your site is live! 📱

> **Updating files?** After re-uploading to S3, go to CloudFront → **Invalidations** tab → Create invalidation → enter `/*` to clear the cache.

---

### Step 6 — (Optional) Custom Domain via Route 53

#### 6a — Buy a Domain

1. Go to **Route 53** → **"Registered domains"** → **"Register domain"**
2. Search for your desired domain name → select it → proceed to checkout
3. Fill in contact details → submit
4. Verify your email when AWS sends a confirmation link
5. Wait 5–30 minutes for activation

#### 6b — Request a Free SSL Certificate (ACM)

> ⚠️ **Must be done in `us-east-1` (US East — N. Virginia) region** — CloudFront only reads certificates from this region.

1. Switch your AWS region to **"US East (N. Virginia)"**
2. Go to **Certificate Manager** → **"Request a certificate"**
3. Select **"Request a public certificate"** → click **"Next"**
4. Add your domain names:
   - `yourdomain.com`
   - `www.yourdomain.com`
5. **Validation method** → select **"DNS validation"**
6. Click **"Request"**
7. Open the certificate → click **"Create records in Route 53"** → confirm
8. Wait 5–10 minutes for status to change to **"Issued"**

#### 6c — Attach Domain to CloudFront

1. Go to **CloudFront** → click your distribution → **"Edit"**
2. Under **"Alternate domain name (CNAME)"** → add:
   - `yourdomain.com`
   - `www.yourdomain.com`
3. Under **"Custom SSL certificate"** → select your certificate from the dropdown
4. Click **"Save changes"** → wait 5–10 minutes to redeploy

#### 6d — Point Domain to CloudFront (Route 53)

1. Go to **Route 53** → **"Hosted zones"** → click your domain
2. Create Record 1 (root domain):
   - **Record name** → leave empty
   - **Record type** → `A`
   - Toggle **"Alias"** ON
   - **Route traffic to** → `Alias to CloudFront distribution`
   - Select your distribution → click **"Create records"**
3. Create Record 2 (www):
   - **Record name** → `www`
   - **Record type** → `A`
   - Toggle **"Alias"** ON
   - **Route traffic to** → `Alias to CloudFront distribution`
   - Select your distribution → click **"Create records"**

✅ Your site is now live at `https://yourdomain.com`

---

## 🔧 Troubleshooting

| Error | Fix |
|---|---|
| **403 Forbidden** | Check S3 bucket policy — make sure it's set correctly (Step 4) |
| **NoSuchKey** | Make sure `index.html` is set as the Default root object in CloudFront |
| **Site not updating** | Create a CloudFront invalidation with `/*` |
| **Certificate not showing in CloudFront** | Make sure ACM certificate was created in `us-east-1` region |
| **Domain not resolving** | DNS propagation can take up to 48 hrs — usually under 1 hr |
| **Deploying stuck 15+ min** | Normal — wait up to 20 mins then refresh |

---

## 💰 Cost Estimate

| Service | Estimated Monthly Cost |
|---|---|
| S3 Storage | ~$0.01 |
| CloudFront | ~$0.10–$0.50 |
| Route 53 Hosted Zone | ~$0.50 |
| Domain (Route 53) | ~$1.00 (amortised) |
| **Total** | **< $2 / month** |

---

## 🗂️ Architecture Overview

```
Mobile / Browser
      │
      ▼
  Route 53 (DNS)
      │
      ▼
  CloudFront (CDN + HTTPS)
      │
      ▼
  S3 Bucket (Static Files)
  ├── index.html
  ├── aws_vpc_deep_dive.html
  ├── aws_security_layers.html
  ├── aws_databases_deep_dive.html
  ├── aws_storage_services.html
  ├── aws_ha_loadbalancing_autoscaling.html
  └── aws_route53_cloudfront_global.html
```

---

## 📄 License

This project is for educational purposes. All AWS service names and logos are trademarks of Amazon Web Services.
