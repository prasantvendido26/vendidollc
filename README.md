# Static Website Deployment – vendidollc.com

This project hosts a static website using:
- AWS S3 for content storage
- CloudFront for global HTTPS delivery
- Route 53 for DNS
- GitHub Actions for CI/CD auto-deployment

---

## Live Site

**URL:** https://vendidollc.com  
**Redirect:** https://www.vendidollc.com → (redirects to non-www)

---

## Architecture Overview

```text
GitHub Repo ──▶ GitHub Actions ──▶ S3 Bucket (vendidollc.com)

CloudFront (HTTPS + Cache + Redirect)
       ▲
Route 53 A Records (www + non-www)
```
---

## S3 Setup

- **Bucket Name:** `vendidollc.com`
- Static website hosting is enabled
  - Index document: `index.html`
- Public access is blocked; content is served through CloudFront
- This is the only bucket used vendidollc.com
- `www.vendidollc.com` is redirected to `vendidollc.com` in CloudFront

---

## CloudFront Setup

- Single CloudFront distribution handles both:
  - `vendidollc.com`
  - `www.vendidollc.com`
- HTTPS enabled using ACM SSL certificate covering both domains
- A **CloudFront Function** named `RedirectWWWToRoot` redirects all traffic from `www.vendidollc.com` to `vendidollc.com`
- Cache is invalidated automatically on each deployment via Github Actions
---

## Route 53 Setup

- Hosted Zone: `vendidollc.com`
- DNS Records:
  - `A` record for `vendidollc.com` → Alias to CloudFront Distribution
  - `A` record for `www.vendidollc.com` → Alias to the same CloudFront Distribution as vendidollc.com

---

## GitHub Actions

- A GitHub Actions workflow auto-syncs all code to the S3 bucket after every push to the `main` branch
- Workflow is defined in `.github/workflows/deploy.yml`
- CloudFront cache is invalidated after each deploy
- No manual upload or invalidation is required

### GitHub Secrets Required

| Secret Name                  | Description                                      |
|------------------------------|--------------------------------------------------|
| `AWS_ACCESS_KEY_ID`          | From IAM user with S3 and CloudFront permissions |
| `AWS_SECRET_ACCESS_KEY`      | From IAM user with S3 and CloudFront permissions |
| `CLOUDFRONT_DISTRIBUTION_ID` | ID of the CloudFront distribution                |

---

## CloudFront Function Logic

The redirect function is attached at the viewer request stage and performs this logic:

- If the `Host` header starts with `www.`
  - Redirects to the same URI on the non-www domain
- Returns `301 Moved Permanently`

Function is managed inside CloudFront console.

---

## Deployment Notes

- Any change to the website needs to be pushed to this Github repository only
- Changes are deployed on every `main` branch push immediately
- Both domains are served securely via CloudFront
  
---
