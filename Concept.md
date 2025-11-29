# Comon CI/CD Attack Vectors
• Supply Chain Attacks
• Code Injection in CI Configuration
• Hardcoded Secrets in Repositories
• Secrets in Pipeline Logs
• Overprivileged Runners and Access Controls

---
## Supply Chain Attacks
- Attackers compromise third-party dependencies used during the build process
- A developer accidentally installs a malicious package (npm install malicious-package)
- The package steals all environment variables (including secrets) and sends them to the attacker's server
- The attacker gains access to secrets and potentially the infrastructure

## Code Injection in CI Configuration
- CI configuration files (e.g., .gitlabci.yml, .github/workflows) can be modified to run malicious commands
- Attacker submits a pull request or gains access to the repository
- Malicious curl command sends sensitive data (/etc/passwd) to an attacker-controlled server
- The malicious code runs during the CI/CD process

## Credential Leaks & Secret Exposure
Secrets (API keys, tokens) accidentally committed to source code can be extracted
Secret is committed to a public or internal repository
• Attackers use tools like truffleHog or GitGuardian to scan for keys
• Compromised keys are used to access cloud infrastructure

*A quick Dorking in GitHub can also get you working credentials*

## Overprivileged Runners and Access Controls
Runners and CI/CD service accounts are often configured with excessive permissions
• If the runner is compromised, the attacker gains access to Docker registry credentials
• Attacker can push a malicious container to the registry.

# Key Security Principles for CI/CD
## Least privilege
• Least Privilege is a security principle where every process, user, or service is given only the minimal set of permissions necessary to perform its function nothing more
• CI/CD pipelines should operate with the least amount of access needed to build and deploy code
• If an attacker gains access to a pipeline with broad permissions, they can compromise infrastructure
• Overprivileged pipelines can accidentally expose sensitive data or allow lateral movement in the network

## Secrets Management
CI/CD pipelines often require sensitive information (API keys, SSH tokens, cloud credentials)
Secrets should never be hardcoded or stored directly in pipeline files
Use a centralized and secure vault to store and manage secrets
If the pipeline logs or config are leaked → the secret is compromised.

## Secure Artifact Storage
Build artifacts (e.g., Docker images, binaries, Helm charts) should be stored in a secure and private repository
• Access to artifacts should be restricted based on roles and environment
• All artifacts should be versioned and immutable after deployment

# CI/CD Security Tools
### SCA (Software Composition Analysis)
• Identifies vulnerabilities in third-party libraries and dependencies.
• Scans open-source components and dependencies in the codebase.
• Checks for known vulnerabilities, outdated libraries, and licensing issues.
• Provides reports on potential security risks and suggests patches or updates

Example Tools:
• Snyk
• Black Duck
• WhiteSource (Now Mend)

### SAST (Static Application Security Testing)
• Finds security flaws in the source code during development
• Analyzes the source code, bytecode, or binaries without executing the application
• Identifies code patterns that may lead to vulnerabilities (e.g., SQL injection, buffer overflow)
• Integrates into CI/CD pipelines to catch issues early

Example Tools:
• Checkmarx
• Veracode
• SonarQube

### DAST (Dynamic Application Security Testing)
• Identifies vulnerabilities during runtime
• Tests the running application from the outside (like a hacker would).
• Sends malicious inputs to simulate real-world attacks.
• Detects issues like XSS (Cross-Site Scripting), CSRF (Cross-Site Request Forgery), and injection attacks.

Example Tools:
• OWASP ZAP
• Burp Suite
• AppSpider

### Secret Scanning
• Detects hardcoded secrets (e.g., API keys, passwords) in code repositories
• Scans for patterns that match sensitive information
• Alerts developers when secrets are exposed
• Can be integrated into CI/CD pipelines

• Example Tools:
• GitGuardian
• TruffleHog
• AWS Secrets Manager

