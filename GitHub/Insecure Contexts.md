
# Insecure Contexts
Contexts are predefined variables that provide access to different types of information
They are structured as JSON objects and can be referenced using **${{ }}** syntax

| Context | Description                                          | Example                             |
| ------- | ---------------------------------------------------- | ----------------------------------- |
| github  | Metadata about the workflow, event, repository, etc. | ${{ github.repository }}            |
| env     | Environment variables defined in the workflow        | ${{ env.MY_VAR }}                   |
| secrets | Repository or organization-level secrets             | ${{ secrets.MY_SECRET }}            |
| jobs    | Information about the jobs within a workflow         | ${{ jobs.job_id.status }}           |
| steps   | Outputs from previous steps                          | ${{ steps.step_id.outputs.result }} |
| runner  | Information about the runner (OS, architecture)      | ${{ runner.os }}                    |
| inputs  | Inputs provided to a workflow                        | ${{ inputs.username }}              |

Insecure Context vulnerabilities arise when user provided input, or uncontrolled data, is used in workflows without proper sanitization.

Insecure values can be:
- Used as arguments to scripts
- Used within run fields
- Passed to other pipelines
- Written to databases

### How Do User Inputs Reach GitHub Actions
Manual Triggers (Using workflow_dispatch)
- GitHub Actions allows you to define a workflow that can be manually triggered by a user
- When a user triggers the workflow, they can provide input values directly through the GitHub UI or API
- The inputs are passed into the workflow using the inputs context

```yml
name: Insecure Context Injection 
 
on: 
  pull_request_target: 
    branches: 
      - main
 
jobs: 
  build: 
    runs-on: ubuntu-latest 
    steps: 
      - uses: actions/checkout@v2 
      - name: Run build 
        run: | 
          echo "Building the project for branch: ${{ github.event.pull_request.head.ref }}"
```

Branch name is a User controlled data and can be manipulated to any value an attacker wants to
`test\"${IFS}&&${IFS}echo${IFS}malicious${IFS}&&${IFS}bash x.sh`, `test\"${IFS}&&${IFS}env&&${IFS}echo${IFS}\"malicious`

#### Attack
- Attacker forks the repo
- Attacker uses a branch name like: `test\"${IFS}&&${IFS}env&&${IFS}echo${IFS}\"malicious`
- Adds some real changes
- Commits to their fork
- Opens a PR that executes arbitrary commands in the runner environment

### Possible Impacts of Insecure Context Vulnerabilities
Arbitrary Code Execution (ACE) 
- A manipulated GitHub Actions input allows running unauthorized shell commands
Repository Takeover
- If a workflow grants excessive permissions (GITHUB_TOKEN with write access), an attacker could push malicious commits
Data Theft & Credential Leakage 
- If a workflow logs secrets (echo ${{ secrets.API_KEY }}), they become visible in logs
Supply Chain Attacks 
- A malicious pull request alters a package before deployment, impacting users
Privilege Escalation 
- A vulnerable workflow running as an admin can be hijacked to modify protected branches
Lateral Movement & Infrastructure Compromise 
- If a workflow contains unvalidated SSH keys or cloud credentials, they can be reused elsewhere

## Mitigating Insecure Context Vulnerabilities
Sanitize User Input Using Env Variables
- **checkout@v3** is pointed at the base branch (main), not the fork.
- **${{ github.event.pull_request.base.ref }}** is trusted input (from the same repo).
- All variables used in run are quoted to avoid shell expansion or injection issues.

Setting Env Variables
- When setting workflow environment variable, GitHub automatically sanitizes the value and removes/escapes common malicious characters
- This renders any malicious payloads inert and will render them as a harmless string

## Best practise beyond insecure context
### Principle of Least Privilege
- Use the least permissions possible for GITHUB_TOKEN
- Avoid giving unnecessary repository or workflow permissions
- Start with read-only, allow write only when necessary
### Avoid pull_request_target for Untrusted Code
- Use pull_request instead to prevent direct execution
- If using pull_request_target, validate inputs before execution
### Mask Secrets & Prevent Leaks
- Never echo secrets or store them in logs!
- Use GitHub’s built-in secret masking
	-  If that value appears in the workflow logs (even in later steps), it will be replaced with *** to prevent accidental exposure.
```
- name: Secure Logging 
- run: echo "::add-mask::${{ secrets.API_KEY }}"
```
- Avoid passing secrets as command arguments use environment variables instead
### Restrict Workflow Permissions
- Set strict permissions to minimize risk
```
permissions:
	actions: read 
	contents: read 
	security-events: write
```
- Disable unnecessary actions to limit attack surface

---
# Attack and Mitigation
## Attack
1. Fork the victim repo
2. Create a commit and push it to a new branch such as `test\"${IFS}&&${IFS}env&&${IFS}echo${IFS}\"malicious`
3. Create a pull request to main and notice your branch name payload is triggered. **but in this case it is within your own repo**
4. Now we attack the victim
	1. Now it should show it is one commit ahead of the victim. 
	2. Click Contribute -> Pull request
	3. ilovetraining8904 wants to merge 1 commit into `wkltraining:main` from `ilovetraining8904:test"${IFS}&&${IFS}env&&${IFS}echo${IFS}"malicious` - Select your malicious branch name
	4. Then create a pull request
	5. Notice your payload is executed within the victim's repo

## Mitigation
```yml
name: Insecure Context Injection Mitigation
 
on: 
  pull_request_target: 
    branches: 
      - main

env:
  BRANCH_NAME: ${{ github.event.pull_request.head.ref }}
 
jobs: 
  build: 
    runs-on: ubuntu-latest 
    steps: 
      - uses: actions/checkout@v2 
      - name: Run build 
        run: | 
          echo "Using environment variables"
          echo "Building the project for branch: $BRANCH_NAME"
```



