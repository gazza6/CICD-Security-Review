How an Attacker Can Exploit It
- Submit Malicious Pull Request
- Drop Payloads on Disk
- Escalate Privileges or Leak Secrets
- Exfiltrate Data or create C2

## Exploitation Scenario
An attacker submits a pull request that modifies the workflow to include malicious code.

If the runner is misconfigured, this code can be executed with the runner's permissions, potentially compromising the system.

## Self-Hosted Runner Exploitation
- Attack targets misconfigured GitHub Actions workflows.
- Exploits a trust boundary in PR workflows and self-hosted runners.
- Common in repos using pull_request_target or head_ref insecurely.

## Step-by-Step Attack Flow
| Step | Description                                                                                       |
| ---- | ------------------------------------------------------------------------------------------------- |
| 1    | A legitimate pull request is submitted (could be a real or staged good-faith PR).                 |
| 2    | The attacker monitors the repo for PR approval events.                                            |
| 3    | Upon approval, attacker gains contributor trust and continues monitoring.                         |
| 4    | The attacker submits another PR with malicious workflow logic (e.g., triggers, unsanitized refs). |
| 5    | This PR appears legitimate by referencing prior contributions.                                    |
| 6    | GitHub Actions automatically triggers the workflow (via pull_request or pull_request_target).     |
| 7    | Workflow checks out the attacker-controlled head_ref without validation.                          |
| 8    | Attacker injects and executes malicious logic via the checked-out branch.                         |
| 9    | Code runs on the self-hosted runner, with potential access to secrets or internal networks.       |
## Critical Mistake – Trusting head_ref
```
- uses: actions/checkout@v3 
  with:
	ref: ${{ github.head_ref }} # This comes from the attacker's fork
```
- This allows the attacker’s code to be executed with base repo privileges
- Often overlooked in pull_request_target or workflow_run scenarios
