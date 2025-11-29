GitHub Actions workflows triggered by **pull_request_target** can be dangerous if not properly configured. One of the most common CI/CD security misconfigurations that attackers exploit to gain unauthorized access and execute arbitrary code

### pull_request vs pull_request_target
**pull_request (Safe)**
- Runs in the context of the forked repository (low privileges)
- Does not have write permissions to the target repository
- Cannot modify secrets from the base repository

**pull_request_target (Dangerous if misconfigured)**
- Runs in the context of the base repository (high privileges)
- Has write access to the repository if configured
- Can access repository secrets

### Why is pull_request_target Dangerous?
If an attacker opens a malicious pull request from a fork, the workflow will execute in the base repository's context, allowing them to steal secrets, modify code, or escalate privileges

## Example
```yml
name: Exploit Pull Request Target
on:
  pull_request_target:
    branches:
      - main

jobs:
  dangerous-job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout PR code
        uses: actions/checkout@v4
        with:
          repository: ${{ github.event.pull_request.head.repo.full_name }}
          ref: ${{ github.event.pull_request.head.ref }}
      - name: Run untrusted code
        run: |
          echo "Running code from untrusted PR..."
          npm install
          npm run build
          npm run test
          
```

### Vuln breakdown here
- Pull Request Target = main
	- This causes workflow to run within the context of the main branch, not the PR branch
- Untrusted Code Execution
	- Potential for malicious activities from PRs, such as secret exfiltration, which could later be used to Privilege Escalation or to move laterally

### Why is this Dangerous?
- Opening a PR containing malicious code in a package.json gets executed against main branch
- Allows for running untrusted code, stealing secrets, etc.

### Why these triggers are being used?
#### pull_request
- Want to test code from external contributors (forks).
- Don’t want to expose secrets or write permissions to untrusted users.
- Common for open-source projects where anyone can open a PR.

#### pull_request_target
- Need to label the PR, post a comment, or run checks that require write access
- Using actions that modify the base repo, like updating commit status or deploying

## Mitigations
- Require Approval: Require approval for external contributors.
- Disable Secrets Access: Disable secrets for `pull_request_target` workflows.
- Sanitize Input: Avoid direct use of `pull_request.head.ref`.
- Restrict Permissions: Limit `GITHUB_TOKEN` permissions.
- Manual Review: Implement manual review steps.
- Prefer `pull_request`: Use `pull_request` over `pull_request_target` when possible
