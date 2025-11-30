## What is Branch Protection?
GitHub’s branch protection rules help enforce best practices, such as requiring:
- Pull request reviews
- Status checks (CI/CD workflows)
- Commit signature verification
- Linear history, and more
- They prevent direct pushes to important branches like main, and enforce peer review and CI before merging a pillar of secure DevOps.
## Bypass
- Craft a malicious PR with GitHub Actions workflow file
- Trigger workflows using pull_request or pull_request_target
- Abuse workflows that check out untrusted code in a privileged context
- Self-approve or auto-merge using GitHub API via GITHUB_TOKEN

## Things needed for a successful attack
- **GITHUB_TOKEN** has write access to contents (merge permission)
- A workflow uses **pull_request_target** or a combination of workflow_run + head_sha
- The PR contains a malicious **.github/workflows/approve.yml** that triggers approval logic.

When a workflow has contents: write permission, it becomes more than just CI it becomes a potential backdoor or malware injector

## Understand the Weapon contents: write
- This permission allows the workflow to:
	- Create or modify any file
	- Commit and push to any branch
	- Overwrite or force-push to main, release, or prod
	- Delete files, folders, or even entire histories
- Now imagine this power triggered by a pull request from an untrusted fork. That's *not* CI, that's remote code execution on your repo

## Pull Request + pull_request_target = Immediate Risk
```
on: pull_request_target 

permissions:
	contents: write
```
- This combo allows attacker-submitted code from a fork to run in the context of your base repo with full write access
- They don’t even need to get merged
### Repo Nuking in Real-Time
- A malicious actor forks your repo and submits a PR like this
- No review. No human approval. Just boom. All files gone from main

## Example
```yml
name: Auto Approve & Merge for Security PRs

on: pull_request_target

permissions:
  contents: read

jobs:
  auto-approve-and-merge:
    if: contains(github.event.pull_request.title, 'security')
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Approve PR
        env:
          GH_TOKEN: ${{ secrets.LAB_BOT_PAT }}
        run: |
          echo "Approving PR #${{ github.event.pull_request.number }}"
          gh pr review ${{ github.event.pull_request.number }} --approve

      - name: Merge PR
        env:
          GH_TOKEN: ${{ secrets.LAB_BOT_PAT }}
        run: |
          echo "Merging PR #${{ github.event.pull_request.number }}"
          gh pr merge ${{ github.event.pull_request.number }} --merge --admin
```

## Mitigation
To prevent this kind of branch protection bypass:
1. Limit token permissions by setting workflow permissions to read-only for pull requests.
2. Set more than one reviewer in branch protection.
3. Disable GitHub Actions from approving PRs in the repository settings under the Actions tab.