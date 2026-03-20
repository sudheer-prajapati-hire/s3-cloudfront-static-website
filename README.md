# S3 + CloudFront Static Website Project


## Live Demo
- CloudFront URL: `https://d1jos70rqafvkk.cloudfront.net`  

---

## Project Overview
This project demonstrates hosting a static website on AWS using **S3, CloudFront, and Origin Access Control (OAC)**.  
It includes versioning, lifecycle policies, private S3 bucket with CloudFront access, and CDN caching.

**Key Features:**
- S3 bucket for static HTML/CSS/JS files
- Versioning enabled for file rollback
- Lifecycle policies to clean old versions automatically
- Private S3 bucket with Origin Access Control (OAC)
- CloudFront distribution for CDN & speed
- HTTPS ready (ACM optional)

---

## AWS Commands Used

### 1. Create S3 Bucket : 
```bash
aws s3api create-bucket \
--bucket sudheer-static-site-12345 \
--region us-east-1

2. Upload Website Files
aws s3 sync ./my-static-website/ s3://sudheer-static-site-12345/ --delete

3. Enable Static Website Hosting 
aws s3 website http://sudheer-devops-site-2026.s3-website-us-east-1.amazonaws.com/ \
--index-document index.html \
--error-document error.html 


4. Enable Versioning
aws s3api put-bucket-versioning \
--bucket sudheer-static-site-12345 \
--versioning-configuration Status=Enabled

5. Set Lifecycle Policy
Create lifecycle.json:

{
  "Rules": [
    {
      "ID": "DeleteOldVersions",
      "Status": "Enabled",
      "Filter": { "Prefix": "" },
      "NoncurrentVersionExpiration": { "NoncurrentDays": 30 }
    }
  ]
}
Apply:
aws s3api put-bucket-lifecycle-configuration \
--bucket sudheer-static-site-12345 \
--lifecycle-configuration file://lifecycle.json 

6. Block Public Access
aws s3api put-public-access-block \
--bucket sudheer-static-site-12345 \
--public-access-block-configuration \
"BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

7. Create Origin Access Control (OAC) for CloudFront
aws cloudfront create-origin-access-control \
--origin-access-control-config \
Name="s3-oac",OriginAccessControlOriginType=s3,SigningBehavior=always,SigningProtocol=sigv4

8. CloudFront Distribution
Create cf-config.json with: 
{
  "CallerReference": "my-static-site-1",
  "Origins": {
    "Quantity": 1,
    "Items": [
      {
        "Id": "S3Origin",
        "DomainName": "sudheer-static-site-12345.s3.amazonaws.com",
        "S3OriginConfig": { "OriginAccessIdentity": "" },
        "OriginAccessControlId": "REPLACE_OAC_ID"
      }
    ]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "S3Origin",
    "ViewerProtocolPolicy": "allow-all",
    "TrustedSigners": { "Enabled": false, "Quantity": 0 },
    "ForwardedValues": { "QueryString": false, "Cookies": { "Forward": "none" } },
    "MinTTL": 0
  },
  "Comment": "Static website without ACM",
  "Enabled": true,
  "DefaultRootObject": "index.html"
}

Create distribution:
aws cloudfront create-distribution --distribution-config file://cf-config.json


9. Invalidate CloudFront Cache
Invalidate all files:
aws cloudfront create-invalidation \
--distribution-id <DISTRIBUTION_ID> \
--paths "/*" 
 

Invalidate a specific file:
aws cloudfront create-invalidation \
--distribution-id <DISTRIBUTION_ID> \
--paths "/index.html" 










