# AWS Static Portfolio

A responsive personal portfolio delivered from a private Amazon S3 bucket through Amazon CloudFront. Terraform manages infrastructure; GitHub Actions validates and scans changes, then deploys website assets using AWS OIDC.

[![CI](https://github.com/mailtosubham25/aws-static-portfolio/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/mailtosubham25/aws-static-portfolio/actions/workflows/ci.yml)
[![Deploy](https://github.com/mailtosubham25/aws-static-portfolio/actions/workflows/deploy.yml/badge.svg?branch=main)](https://github.com/mailtosubham25/aws-static-portfolio/actions/workflows/deploy.yml)

## Live site

[Visit the portfolio](https://d2ymfb3hoeygmq.cloudfront.net)

## Project highlights

- Private S3 origin with all S3 public-access-block settings enabled.
- CloudFront Origin Access Control (OAC), HTTPS redirect, and an AWS-managed response headers policy.
- Terraform for the S3 bucket, CloudFront distribution, access policy, versioning, encryption, and lifecycle retention.
- GitHub Actions CI for Terraform formatting/validation and Checkov and Trivy security scans.
- GitHub Actions deployment to S3 via OIDC; website assets are uploaded and the CloudFront cache is invalidated after pushes to `main`.
- A security review register documents policy exceptions and review triggers.

## Architecture

~~~mermaid
flowchart LR
    Developer[Developer] -->|Pull request or push| GitHub[GitHub repository]
    GitHub --> CI[CI workflow]
    CI --> Validate[Terraform fmt and validate]
    CI --> Checkov[Checkov baseline gate and full report]
    CI --> Trivy[Trivy config and secret scans]

    GitHub -->|Push to main| Deploy[Deploy workflow]
    Deploy -->|OIDC role assumption| Role[AWS deploy role]
    Role -->|Upload index, CSS, images| S3[(Private S3 bucket)]
    Role -->|Invalidate cache| CF[CloudFront distribution]

    Visitor[Visitor] -->|HTTPS| CF
    CF -->|Signed origin requests with OAC| S3
    Terraform[Terraform configuration] --> S3
    Terraform --> CF
~~~

## Security checks

The CI workflow blocks regressions in selected baseline controls for S3 encryption, versioning and public-access protection, CloudFront HTTPS, OAC, and response headers. The full Checkov scan remains visible as a report. Trivy blocks new HIGH or CRITICAL infrastructure findings and detected secrets; the WAF and customer-managed KMS findings have time-limited documented exceptions in `.trivyignore`.

See the [security review and remediation register](docs/security-review.md) for current findings, rationale, cost considerations, and review triggers.

## Deploying changes

- Open a pull request to run Terraform validation and security checks.
- Merge to `main` to trigger the website deployment workflow. It assumes the configured AWS role with GitHub OIDC, uploads `index.html`, `css/`, and `images/`, then invalidates CloudFront.
- Terraform provisioning is separate from website publishing. Read [deployment instructions](docs/deployment.md) before changing AWS resources. Terraform state is local; do not apply from a clean checkout without the existing state or a deliberate import/recovery plan.

## Repository layout

~~~text
.github/workflows/ci.yml       Validation and security scans
.github/workflows/deploy.yml   Website deployment through OIDC
css/                           Stylesheet
docs/deployment.md             Provisioning and publishing instructions
docs/security-review.md        Findings, exceptions, and review triggers
images/                        Profile photo
index.html                     Portfolio content
infrastructure/                Terraform for S3 and CloudFront
~~~

## Résumé-ready summary

Designed and deployed a static AWS portfolio using Terraform, a private S3 origin, CloudFront OAC, HTTPS, and response security headers. Added GitHub Actions CI/CD with Terraform validation, Checkov and Trivy scans, and OIDC-based website deployment.

## Author

**Shubham Mandal** — AWS Cloud & DevOps Engineer

[GitHub profile](https://github.com/mailtosubham25)
