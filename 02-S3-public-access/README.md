# 🪣 AWS S3 — Public Access & Bucket Policy

> Making S3 objects publicly accessible following AWS best practices.

---

## 🗺️ Mind Map
```
S3 Public Access
├── 🔓 Block Public Access
│   └── Must disable all 4 settings first
│
├── 📋 Bucket Policy
│   ├── AWS default = deny everything
│   └── Must explicitly allow what you need
│
├── 🌐 URL Types
│   ├── Object URL   → direct file access
│   └── Website URL  → static website hosting
│
└── ⚠️ Common Mistakes
    ├── Using wrong region in URL
    ├── Block public access still ON
    └── Missing bucket policy
```

---

## 💡 How S3 Access Works
```
Incoming Request
       │
       ▼
Block Public Access ON?──── YES ──→ ❌ Denied
       │
      NO
       │
       ▼
Bucket Policy exists? ───── NO ───→ ❌ Denied (AWS default)
       │
      YES
       │
       ▼
Policy allows action? ───── NO ───→ ❌ Denied
       │
      YES
       │
       ▼
       ✅ Allowed
```

> AWS denies everything by default.
> You must explicitly allow access.

---

## 📋 Step 1 — Disable Block Public Access
```bash
# Check current status
aws s3api get-public-access-block \
  --bucket YOUR-BUCKET \
  --profile YOUR-PROFILE
```

All 4 values must be `false`:
```json
{
    "PublicAccessBlockConfiguration": {
        "BlockPublicAcls": false,
        "IgnorePublicAcls": false,
        "BlockPublicPolicy": false,
        "RestrictPublicBuckets": false
    }
}
```

---

## 📋 Step 2 — Add Bucket Policy
```bash
# Check if policy exists
aws s3api get-bucket-policy \
  --bucket YOUR-BUCKET \
  --profile YOUR-PROFILE

# Add public read policy
aws s3api put-bucket-policy \
  --bucket YOUR-BUCKET \
  --profile YOUR-PROFILE \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET/*"
    }]
  }'
```

---

## 📋 Step 3 — Verify & Test
```bash
# Verify bucket region
aws s3api get-bucket-location \
  --bucket YOUR-BUCKET \
  --profile YOUR-PROFILE

# Test object access
curl -I https://YOUR-BUCKET.s3.REGION.amazonaws.com/YOUR-FILE
```

Expected:
```
HTTP/1.1 200 OK ✅
```

---

## 🌐 S3 URL Reference

| Type | Format | Use Case |
|------|--------|----------|
| Object URL | `https://BUCKET.s3.REGION.amazonaws.com/FILE` | Direct file access |
| Website URL | `http://BUCKET.s3-website-REGION.amazonaws.com` | Static website |

---

## ✅ Best Practices
```
✅ Disable block public access before adding policy
✅ Use bucket policy — never make bucket public via ACL
✅ Scope policy to specific bucket ARN — not all buckets
✅ Verify correct region in URL
✅ Use CloudFront + OAC for production — keep S3 private
```

---

## ⚠️ Public Access vs Production

| Use Case | Method |
|----------|--------|
| Learning / testing | Public bucket policy ✅ |
| Production website | CloudFront + OAC (S3 stays private) ✅ |

> Never expose S3 directly in production.
> Always use CloudFront as the front door.

---

*Author: KO KO SHINE — March 2026*
