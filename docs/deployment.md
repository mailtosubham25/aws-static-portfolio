# Deployment instructions

## Website publishing

The live site is served from CloudFront: https://d2ymfb3hoeygmq.cloudfront.net.

1. Create a branch and update website files.
2. Open a pull request to `main`. CI checks Terraform formatting/validation, runs the selected blocking Checkov baseline and full report, and runs Trivy infrastructure and secret scans.
3. Merge after reviewing the checks.
4. The deploy workflow assumes `GitHubActionsPortfolioDeployRole` through GitHub OIDC, uploads `index.html`, `css/`, and `images/` to the private S3 bucket, then invalidates CloudFront.
5. Confirm the Actions deploy run succeeds and verify the CloudFront URL.

The deploy workflow does not publish the repository root. `README.md`, `.gitignore`, Terraform files, and docs are not copied to S3.

## Infrastructure changes

Terraform configuration is in `infrastructure/environments/prod` and targets `ap-south-1`. The website deployment workflow does not run `terraform apply`.

The Terraform configuration has no remote backend, so state is local and ignored by Git. Before running Terraform against the existing account:

- Use the original state file or deliberately import deployed resources into a recovered state.
- Back up state securely; never commit state or plan files.
- Run `terraform init`, inspect `terraform plan`, and apply only reviewed changes.

A clean checkout without existing state will treat deployed resources as unmanaged and may propose duplicates or fail on existing names. Do not apply such a plan.

## Verification

After publishing, check that the CloudFront URL returns HTTP 200 over HTTPS and includes the configured security headers. The CI and deploy workflows are available in the repository Actions tab.
