What is CodeBuild?
	- Fully managed CI/CD service
	- Compiles source code, runs tests, produces artifacts
- Part of AWS Developer Tools suite
- Commonly integrated with CodePipeline, CodeCommit, GitHub

## How CodeBuild Works
- Pulls code from:
    - S3, CodeCommit, GitHub, Bitbucket, etc.
- Uses buildspec.yml to define build commands
- Builds inside an ephemeral container
- Can access VPC, IAM roles, secrets, ECR, SSM

Anatomy of a CodeBuild Project
- Source (repo, branch)
- Environment (Docker image, compute size)
- Buildspec file
- Artifacts (output)
- IAM Role (assumes permissions to AWS)

## Example - Secure IAM Role
Avoid granting sts:AssumeRole unless necessary

## buildspec.yml
- Can define multiple phases: install, pre_build, build, post_build
- Danger: arbitrary command execution

### Attack Scenario – Malicious PR
- CodeBuild project is triggered on pull requests
- buildspec.yml is attacker-controlled
- If CodeBuild has access to AWS, attacker can exfiltrate data

### Privilege Escalation Vector
- CodeBuild with:
	- ssm:GetParameter, kms:Decrypt, sts:AssumeRole
- Attacker injects payload to:
	- Access secrets
	- Escalate via sts:AssumeRole into prod roles

### Artifact Poisoning in CodeBuild
Artifacts can be stored in:
- S3
- CodePipeline

### Secrets Leakage Risk
- CodeBuild commonly pulls secrets from:
	- SSM Parameter Store
	- Secrets Manager
- If attacker gains code execution:
	- Extract secrets
	- Move laterally into more sensitive areas (RDS, Lambda, EC2)

## Exploitation Risks in AWS CodeBuild
Potential risks when using AWS CodeBuild include:
- Over-permissive service roles accessing S3, Secrets Manager, etc.
- STS token leakage allowing unauthorized API access
- Forked PR builds running untrusted code
- Increased S3 costs due to malicious uploads