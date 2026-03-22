# ⚙️ AWS CLI Setup on Ubuntu

> Phase 01 — Local machine to AWS connection

---

## 🗺️ Mind Map
```
AWS CLI Setup
├── 📦 Installation
│   ├── OS: Ubuntu 24
│   ├── Method: curl + unzip
│   └── Version: aws-cli/2.22.7
│
├── 👤 IAM Configuration
│   ├── Create IAM User
│   │   ├── Programmatic access only
│   │   └── No console access
│   ├── Create Custom Policy
│   │   └── Least privilege (scoped to bucket ARN)
│   └── Create IAM Group
│       ├── Attach policy to group
│       └── Add user to group
│
├── 🔑 Access Keys
│   ├── One key per purpose
│   ├── Add tags for description
│   └── Store in ~/.aws/credentials
│
└── ⚙️ CLI Configuration
    ├── Named profiles (not default)
    ├── Region: ap-southeast-1
    └── Format: json
```

---

## 👤 Step 1 — Create IAM User (AWS Console)
```
AWS Console → IAM → Users → Create user
```

| Field | Value |
|-------|-------|
| Username | `koko-s3-user` |
| Console access | ❌ Disable |
| Programmatic access | ✅ Enable |

---

## 🔐 Step 2 — Create IAM Group & Policy

**Create Group:**
```
IAM → User groups → Create group
Name: s3-deploy-group
```

**Create Custom Policy:**
```
IAM → Policies → Create policy → JSON
```
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBuckets",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    },
    {
      "Sid": "S3BucketAccess",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": "arn:aws:s3:::your-bucket-name"
    },
    {
      "Sid": "S3ObjectAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

**Attach Policy to Group:**
```
IAM → User groups → s3-deploy-group
→ Permissions → Attach policy → koko-s3-policy
```

**Add User to Group:**
```
IAM → User groups → s3-deploy-group
→ Users → Add users → koko-s3-user
```

---

## 🔑 Step 3 — Generate Access Key
```
IAM → Users → koko-s3-user
→ Security credentials
→ Access keys → Create access key
→ Use case: CLI
→ Download CSV ← save this!
```

> ⚠️ Secret key is shown **only once** — save CSV immediately.

---

## 📦 Step 4 — Install AWS CLI
```bash
# Download
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Unzip
unzip awscliv2.zip

# Install
sudo ./aws/install

# Verify
aws --version
# Expected: aws-cli/2.22.7 Python/3.12.6 Linux/x86_64
```

---

## ⚙️ Step 5 — Configure Named Profile
```bash
aws configure --profile koko-s3-user
```

Fill in the prompts:
```
AWS Access Key ID:     AKIA...        ← from CSV
AWS Secret Access Key: xxxxxxxx       ← from CSV
Default region name:   ap-southeast-1
Default output format: json
```

---

## 📁 Step 6 — Verify Credentials & Config Files
```bash
# View credentials file
cat ~/.aws/credentials
```

Expected output:
```ini
[koko-s3-user]
aws_access_key_id = AKIA...
aws_secret_access_key = xxxxxxxx
```
```bash
# View config file
cat ~/.aws/config
```

Expected output:
```ini
[profile koko-s3-user]
region = ap-southeast-1
output = json
```

---

## ✅ Step 7 — Test Connection
```bash
# Verify identity
aws sts get-caller-identity --profile koko-s3-user
```

Expected output:
```json
{
    "UserId": "AIDA...",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/koko-s3-user"
}
```
```bash
# Set profile for current session
export AWS_PROFILE=koko-s3-user

# Test S3 access
aws s3 ls
```

---

## ✅ Best Practices
```
✅ One IAM user per service/purpose
✅ Use named profiles — not [default]
✅ Least privilege — scope to specific bucket ARN
✅ Add tags to access keys for description
✅ Never commit ~/.aws/credentials to GitHub
✅ Use IAM Groups to manage permissions
✅ Programmatic access only — no console login
```

---

## ⚙️ Environment

| Item | Value |
|------|-------|
| OS | Ubuntu 24 |
| AWS CLI | v2.22.7 |
| Python | 3.12.6 |
| Region | ap-southeast-1 |
| Profile | koko-s3-user |

---

*Author: KO KO SHINE — March 2026*
