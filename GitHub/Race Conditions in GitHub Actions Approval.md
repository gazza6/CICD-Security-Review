- A race condition occurs when the outcome of a process depends on the timing of uncontrollable events
- In the context of GitHub Actions, it means an attacker exploits the delay between a workflow being approved and the workflow actually starting

A race condition can occur when:
- A workflow is approved, but the code it runs can be changed before execution
- Two systems or processes try to write or read shared data at the same time
- Something assumes the state hasn’t changed, when it actually has (e.g., HEAD in Git has moved)

## In GitHub Actions Context
A race condition in GitHub Actions happens when two or more events compete in time, and the order in which they happen determines whether the system behaves safely or gets exploited
- In GitHub Actions, this usually refers to the time gap between:
- A workflow being approved (e.g., by a maintainer), and
- The workflow being executed, especially when it’s based on untrusted pull request code

Imagine this:
- You ask your friend to review a document before printing
- While they’re approving it, you swap the final page with something inappropriate
- They approved it thinking it was clean but what prints is not what they reviewed
- That’s exactly what an attacker does in GitHub Actions when exploiting a race condition

That’s exactly what an attacker does in GitHub Actions when exploiting a race condition.

## Why It’s Dangerous in GitHub Actions
- GitHub Actions allows workflows to be triggered from forked repositories.
- For security, some workflows (like pull_request_target) require manual approval before secrets or privileged access is granted
- But here's the problem:
- An attacker can submit a clean pull request, wait for it to be approved, and then quickly push malicious changes before the workflow starts running.
- This means the workflow runs with trusted permissions but now on attacker-controlled code.

## Attack
Step 1 – Submit a “safe-looking” PR
- Submit a PR with something innocent like fixing a typo.
- It targets a workflow that uses pull_request_target which means it runs with base repo permissions after approval

Step 2 – Wait for approval
- Maintainers review the PR and approve it.
- GitHub marks it as approved and prepares to start the workflow

Step 3 – Quickly push a malicious commit
- Before the workflow actually starts (there’s a small delay), you forcepush new code to the same branch
- Now the latest commit includes your payload but GitHub still thinks it’s running trusted code

Step 4 – Workflow runs with elevated privileges
- The pull_request_target workflow or workflow_run starts.
- It checks out ${{ github.head_ref }} which now contains malicious code.
- It executes that code on a self-hosted runner or with access to secrets and write tokens.

## Why It’s a Race
- GitHub doesn’t lock the branch after approval
- There's no binding between the commit that was approved and the one executed unless you explicitly enforce it
- The attacker races to change the code between approval and execution

Vulnerable Example:
```yml
on:
  pull_request_target:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.head_ref }}   # attacker-controlled branch

```

If this workflow runs after approval, it checks out whatever code is currently in that branch, even if it changed after approval.

## Attack Path – Step-by-Step
| Step                          | Description                                                                                                     |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------- |
| 1. Legitimate contribution PR | Attacker submits a genuine-looking PR that appears harmless to reviewers.                                       |
| 2. Monitor approval           | Attacker monitors for approval events using GitHub notifications, bots, or polling.                             |
| 3. Push malicious code        | Right after approval (but before workflow runs), attacker force-pushes a commit with malicious changes.         |
| 4. DEV (maintainer)           | Maintainer approves the PR, unaware of last-second changes.                                                     |
| 5. Malicious changes inserted | The base branch appears unchanged, but head_ref now points to attacker’s payload.                               |
| 6. PR is marked “Approved”    | The PR is trusted, and workflows run under elevated or self-hosted runner contexts.                             |
| 7. Pipeline is triggered      | Workflow starts based on PR event uses pull_request_target or workflow_run.                                     |
| 8. Checkout of head_ref       | Workflow checks out the attacker’s malicious branch.                                                            |
| 9. Execution occurs           | Self-hosted runner executes the payload under trusted permissions (may access secrets, internal systems, etc.). |

## Mitigations for head_ref in GitHub Actions- Avoid Using github.head_ref in Privileged Workflows
- Use github.event.pull_request.base.ref Instead
- Use pull_request Instead of pull_request_target
- Avoid Running Scripts or Build Steps Based on Forked PRs in Privileged Context
- Use if Conditions to Validate Source
- Pin Actions and Use Checksums
- Use pull_request Instead of pull_request_target
- Never Trust github.head_ref Directly
- Lock the Commit SHA at the Time of Approval: `ref: ${{ github.event.pull_request.head.sha }}`
    - And add a guard: `"$GITHUB_SHA" != "$(git rev-parse HEAD)“`