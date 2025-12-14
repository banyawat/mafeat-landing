# Deployment Setup

This workflow uses **OIDC (OpenID Connect)** for secure authentication with AWS - no long-lived credentials needed!

## Why OIDC?

Benefits over traditional AWS access keys:
- **No long-lived credentials** - temporary tokens expire automatically
- **Better security** - no secrets to rotate or accidentally leak
- **Least privilege** - restrict access to specific repos and branches
- **Audit trail** - AWS CloudTrail shows which GitHub workflow assumed the role
- **No key rotation** - GitHub and AWS handle token exchange automatically

## AWS OIDC Setup

### Step 1: Create OIDC Identity Provider in AWS

1. Go to AWS IAM Console > **Identity providers** > **Add provider**
2. Configure the provider:
   - **Provider type:** OpenID Connect
   - **Provider URL:** `https://token.actions.githubusercontent.com`
   - **Audience:** `sts.amazonaws.com`
3. Click **Add provider**

### Step 2: Create IAM Role for GitHub Actions

1. Go to IAM Console > **Roles** > **Create role**
2. Select **Web identity**:
   - **Identity provider:** token.actions.githubusercontent.com
   - **Audience:** sts.amazonaws.com
3. Click **Next**
4. Attach the policy (see below)
5. Name the role (e.g., `GitHubActionsDeployRole`)
6. **Before creating**, edit the trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/mafeat-landing:*"
        }
      }
    }
  ]
}
```

Replace:
- `YOUR_ACCOUNT_ID` with your AWS account ID
- `YOUR_GITHUB_USERNAME/mafeat-landing` with your GitHub repo (e.g., `octocat/mafeat-landing`)

7. Create the role and copy the **Role ARN** (you'll need this for GitHub secrets)

### Step 3: Create and Attach IAM Policy

The IAM role needs the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation",
        "cloudfront:GetInvalidation"
      ],
      "Resource": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DISTRIBUTION_ID"
    }
  ]
}
```

Replace:
- `YOUR_BUCKET_NAME` with your S3 bucket name
- `YOUR_ACCOUNT_ID` with your AWS account ID
- `YOUR_DISTRIBUTION_ID` with your CloudFront distribution ID

Create this policy in IAM and attach it to the role created in Step 2.

## GitHub Secrets Configuration

Add these secrets to your GitHub repository:

1. Go to your repository on GitHub
2. Navigate to **Settings** > **Secrets and variables** > **Actions**
3. Click **New repository secret** and add the following:

### Required Secrets

| Secret Name | Description | Example |
|------------|-------------|---------|
| `AWS_ROLE_ARN` | IAM role ARN for OIDC authentication | `arn:aws:iam::123456789012:role/GitHubActionsDeployRole` |
| `AWS_REGION` | AWS region where your S3 and CloudFront are | `us-east-1` |
| `S3_BUCKET_NAME` | S3 bucket name for static hosting | `mafeat-landing` |
| `CLOUDFRONT_DISTRIBUTION_ID` | CloudFront distribution ID | `E1234567890ABC` |

## How to Deploy

1. **Create and push a tag:**
   ```bash
   git tag 1.0.0
   git push origin 1.0.0
   ```

2. **Or create a tag with annotation:**
   ```bash
   git tag -a 1.0.0 -m "Release version 1.0.0"
   git push origin 1.0.0
   ```

3. **GitHub Actions will automatically:**
   - Build the static site using `npm run build:production`
   - Upload files to S3 with optimized cache headers
   - Invalidate CloudFront cache
   - Show deployment summary

## Cache Strategy

The workflow uses a smart caching strategy:
- **Static assets** (JS, CSS, images): 1 year cache (`max-age=31536000`)
- **HTML/XML/TXT files**: No cache (`max-age=0, must-revalidate`)

This ensures users always get the latest content while benefiting from cached assets.

## Monitoring Deployments

- View workflow runs: **Actions** tab in your GitHub repository
- Each deployment shows a summary with tag, bucket, and distribution info
- Failed deployments will show detailed error logs

## Rollback

To rollback to a previous version:
```bash
# Find the previous tag
git tag -l

# Create a new tag pointing to the old commit
git tag 1.0.1 <old-commit-hash>
git push origin 1.0.1
```

## Troubleshooting

**Deployment fails with "Access Denied" or "Not authorized to perform sts:AssumeRoleWithWebIdentity":**
- Verify the trust policy in your IAM role matches your GitHub repo exactly
- Check that the OIDC provider is configured correctly in AWS
- Ensure `AWS_ROLE_ARN` secret contains the full role ARN
- Verify the IAM policy is attached to the role

**Error: "An error occurred (AccessDenied) when calling the PutObject operation":**
- Check S3 bucket name is correct in secrets
- Verify IAM policy has s3:PutObject permission
- Ensure the bucket exists and is in the correct region

**Files not updating on CloudFront:**
- Wait a few minutes for invalidation to complete
- Check CloudFront invalidation status in AWS Console
- Verify `CLOUDFRONT_DISTRIBUTION_ID` is correct

**Build fails:**
- Check Node.js version compatibility
- Verify all dependencies are in package.json
- Review build logs in GitHub Actions for specific errors

**OIDC token exchange fails:**
- Ensure the workflow has `permissions: id-token: write` set
- Verify the OIDC provider thumbprint is correct in AWS
- Check that the repository is not private (or OIDC is configured for private repos)
