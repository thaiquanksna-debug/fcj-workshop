---
title: "Deploy Static Website on AWS with Amazon S3 Private, CloudFront and OAC"
date: 2026-07-05
weight: 1
chapter: false
pre: " <b> 3.1 Blog 1. </b> "
---


## Introduction

When starting with AWS, many people often deploy static websites by enabling **Static Website Hosting** on Amazon S3, uploading HTML, CSS, and JavaScript files to the bucket, and then allowing public access.

This approach is highly suitable for learning or demo purposes because it is simple and easy to implement. However, when moving closer to a production environment, making an S3 bucket directly public is not always the optimal choice.

In practice, a more common architecture involves:

- Amazon S3 Private Bucket
- Amazon CloudFront
- Origin Access Control (OAC)

This model separates the data storage layer from the content delivery layer, while improving security and scalability.

---

# Architecture Overview

The system traffic flow follows this sequential order:

- **User** sends requests via **HTTPS** to **CloudFront**.
- **CloudFront** performs an **OAC Signed Request** to fetch data from the **Private S3 Bucket**.

Users only access CloudFront.

CloudFront requests data from S3 on behalf of the user via Origin Access Control.

In this model:

- CloudFront acts as the public entry point
- S3 serves solely for data storage
- Users cannot directly access the bucket

---

# Core Components

## Amazon S3

Amazon S3 is responsible for storing all static files:

- index.html
- JavaScript
- CSS
- Images
- Fonts
- Other static assets

Recommended configurations to apply:

- Block Public Access
- Bucket Owner Enforced
- Keep Static Website Hosting disabled
- Allow only CloudFront to read objects

This helps reduce the risk of direct access from the Internet.

---

## Amazon CloudFront

CloudFront acts as the CDN and serves as the public entry point for the website.

Key benefits include:

- HTTPS
- CDN Cache
- Custom Domain
- HTTP → HTTPS Redirect
- Global Edge Locations
- Security Headers
- Logging

CloudFront helps reduce latency and speeds up page loading for users across different regions.

---

## Origin Access Control (OAC)

Origin Access Control allows CloudFront to access S3 securely.

With OAC:

- CloudFront signs requests sent to S3
- The Bucket Policy only permits the specified CloudFront Distribution to access it

This is much better than making the bucket public.

An important note to consider is that OAC only works with **S3 REST Endpoints** and does not function with **S3 Website Endpoints**.

---

# Security and the Principle of Least Privilege

The Bucket Policy should only allow the `s3:GetObject` action and apply specifically to:

- Static website files
- The designated CloudFront Distribution

However, it is vital to understand that:

OAC is not a file-level authorization mechanism.

If a file exists in the bucket and CloudFront has read permissions, that file can still be accessed if the user knows the exact URL.

Therefore:

- Do not store internal documents in the web bucket
- Do not store backups
- Do not store sensitive data

You should use a dedicated bucket for website artifacts.

---

# Cache Strategy

One of the most common issues is: **Uploading a new version, but the website still displays the old version**.

This is usually because CloudFront or the browser is caching the old object.

---

## Using Hashed Filenames

Modern frameworks such as React, Vue, Angular, or Vite typically generate files with attached hash strings, for example: `app.8f3a1c.js` or `style.92ac0d.css`.

With every build:

- Filenames change
- The old cache does not affect the updates

Consequently, you can cache assets for a long duration.

---

## Caching for Assets

For example, you can apply the response header configuration: `Cache-Control: public, max-age=31536000, immutable`

This should be applied to:

- JS
- CSS
- Images
- Fonts

---

## Caching for index.html

Conversely, use: `Cache-Control: no-cache, no-store, must-revalidate` or configure a short TTL.

Reason: index.html must always point to the latest version of the application.

---

## CloudFront Invalidation

Upon deployment:

1. Upload hashed assets
2. Upload index.html
3. Invalidate `/index.html`

You should target the exact path `/index.html`. Avoid overusing global invalidations like `/*` because it increases costs, slows down cache propagation, and is sub-optimal.

---

# SPA Routing

Single Page Applications (SPAs) often use virtual routes like `/dashboard`, `/profile`, or `/settings`. However, the corresponding files or folders do not actually exist in the S3 bucket.

When a user refreshes the page at `/dashboard`, CloudFront might return a `403` or `404` error code.

---

## Common Solutions

Configure a Custom Error Response to forward errors from `403 → /index.html` and `404 → /index.html`, while returning an `HTTP 200` status code.

This enables routing libraries like React Router, Vue Router, or Angular Router to handle the navigation properly on the client side.

---

## Notes

If you redirect all errors to index.html:

- It might mask actual errors
- It makes debugging harder

A better approach is to:

- Rewrite routes that do not have file extensions
- Keep requests intact for JS, CSS, PNG, SVG

CloudFront Functions are commonly used for this purpose.

---

# Custom Domain and SSL

When using CloudFront with a custom domain, the certificate in AWS Certificate Manager (ACM) must reside strictly within the **us-east-1 (N. Virginia)** region.

This is a very common mistake for beginners. For example, a faulty configuration would be:

- Bucket in Singapore
- Certificate in Singapore

As a result, CloudFront will not be able to detect or utilize that certificate.

---

# Common Troubleshooting

## 403 Access Denied

Make sure to double-check:

- The Bucket Policy
- Whether OAC is attached properly
- If the origin points to the correct REST Endpoint
- If the requested object actually exists
- If the path matches exact case-sensitivity

---

## Website Still Displays the Old Version

Verify the following header values: Cache-Control, Age, X-Cache, ETag, and Last-Modified.

---

## Custom Domain Not Working

Verify that:

- ACM is located in us-east-1
- The certificate has been successfully validated
- Alternate Domain Names (CNAMEs) are set up in CloudFront
- DNS Records are properly configured at your domain registrar

---

# Advantages of the Architecture

- S3 remains private
- HTTPS by default
- Global CDN
- Support for custom domains
- Efficient cache management
- Easily scalable with WAF and Monitoring

---

# Certain Limitations

- More complex configuration
- Harder to debug
- Incurs CloudFront costs
- Requires a clear understanding of Bucket Policy and OAC

---

# Next Steps for Development

After a successful deployment via the AWS Console, the next logical step is transitioning to Infrastructure as Code.

Popular choices include:

- Terraform
- AWS CDK
- CloudFormation

You can manage the entire stack—S3 Bucket, CloudFront Distribution, OAC, Bucket Policy, Cache Behavior, and Custom Domain—using source code instead of manual operations.

---

# CI/CD Integration

A straightforward deployment pipeline may include the following sequential steps:

1. Build the frontend
2. Upload assets to S3
3. Upload index.html
4. Trigger CloudFront Invalidation
5. Execute a Smoke Test

Popular tools: GitHub Actions, GitLab CI/CD, and AWS CodePipeline.

---

# Conclusion

Hosting a static website with Amazon S3 is an excellent practice for AWS beginners.

However, when aiming for a production-ready environment, an architecture using an Amazon S3 Private Bucket, Amazon CloudFront, and Origin Access Control becomes the more appropriate choice.

This model enhances security, optimizes performance, efficiently manages caching, and builds a solid foundation to scale further into Infrastructure as Code, CI/CD, Monitoring, and real-world operations.

<h2 style="text-align:center;">Architecture Diagram</h2>

![Photo](/images/3-Blog/diagram1.jpg)

**Article Link** : https://www.facebook.com/groups/awsstudygroupfcj/permalink/2180621306036163