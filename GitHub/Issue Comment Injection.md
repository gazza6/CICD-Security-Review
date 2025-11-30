### What is Issue Comment Injection?
- Happens when a GitHub Actions workflow is triggered by issue_comment.
- Attacker crafts malicious input in an issue or PR comment.
- If workflow parses or executes the comment content without validation, it can lead to:
    - Command injection
    - Information disclosure
    - Arbitrary behavior

### Understanding Issue Comment Injection
- The `issue_comment` event can cause injection vulnerabilities
- The GitHub token permissions depend on repo settings

```yml
on:
	issue_comment:
		types: [created]

jobs:
	handle-comment:
		runs-on: ubuntu-latest 
		steps:
			- name: Run comment as command (bad!)
			run: ${{ github.event.comment.body }}
```
If someone comments “rm -rf /” this will literally execute that command
#### Vulnerability Breakdown
- Code Execution: Arbitrary commands can be executed
- Data Exfiltration: Attacker can access secrets or sensitive information
- If the repository checks out the PR, code execution is possible (like IPPE)

## Attack
```yml
name: Issue Comment Injection Workflow 
 
on: 
  issue_comment: 
    types: [created] 
 
jobs: 
  comment-job: 
    runs-on: ubuntu-latest 
    if: contains(github.event.comment.body, '/benchmark')
    steps: 
      - name: Checkout the repository 
        uses: actions/checkout@v4 
      - name: Run bash command using comment body 
        run: | 
          comment_body="${{ github.event.comment.body }}" 
          echo "The comment body contains: $comment_body"
```

### Workflow logic
This workflow is triggered whenever a new comment is created on an issue within the repository. It executes within an Ubuntu-based runner and includes a job that only runs if the comment body contains the string */benchmark*. Within the run step, the comment text is stored in a variable named *comment_body*, which is then echoed to the output. However, because the *github.event.comment.body* variable is used directly without proper sanitization, this creates a potential security risk. If an attacker injects malicious content into the comment, it could be executed, making the workflow vulnerable to command injection.

### Steps
1. Create an issue
2. Leave a comment with `/benchmark" && env && echo "hacked`
3. Check github actions

## Mitigations
**Strict Command Whitelisting**
- Only allow specific commands like /deploy, /approve, /test.
- Use conditional logic: `if [[ "$COMMENT" == "/deploy" ]]; then ./deploy.sh; fi`
**Avoid Direct Execution of Comments**
- Never run “eval ${{ github.event.comment.body }}” or similar.
- Parse and sanitize inputs before using them.
**Input Validation & Escaping**
- Use regex to extract and validate the command: `[[ "$COMMENT" =~ ^/(deploy|test|approve)$ ]] || exit 1`
**Use GitHub’s Permission Model**
- Only trigger workflows on comments from trusted users
- Use “github.event.comment.user.login” to filter by allowed usernames or org members
**Avoid Secrets in Comment-Based Workflows**
- Do not expose secrets in jobs that process external comments
- Use “pull_request” instead of “issue_comment” if secrets aren't required.

**Example mitigation by using env**
```yml
name: Mitigated Issue Comment Injection Workflow 
 
on: 
  issue_comment: 
    types: [created]

env:
  COMMENT_BODY: ${{ github.event.comment.body }}
 
jobs: 
  comment-job: 
    runs-on: ubuntu-latest 
    if: contains(github.event.comment.body, '/benchmark')
    steps: 
      - name: Checkout the repository 
        uses: actions/checkout@v4 
      - name: Run bash command using comment body 
        run: | 
          echo "The comment body contains: $COMMENT_BODY"
```