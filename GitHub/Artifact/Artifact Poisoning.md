Overview
Artifact Poisoning is when an attacker:
- Injects a malicious file or payload into an artifact (e.g., test results, build output)
- That artifact is then used by another trusted workflow, giving the attacker's code elevated access (like secrets or write permissions)

This module demonstrates how misconfigured GitHub Actions workflows can lead to artifact poisoning attacks.

Key Points:
- Exploit file overwriting behavior
- Privilege escalation
- Secure workflows to avoid this

## Mitigation Strategies
To prevent artifact poisoning:
- Specify download paths to avoid overwrites
- Limit token permissions
- Ensure proper review of artifacts and workflows