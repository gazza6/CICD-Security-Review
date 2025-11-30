What is Dependabot?
- Dependabot is a GitHub feature that automatically creates pull requests (PRs) to update dependencies in your project.
- It helps keep libraries and packages up to date, especially when security patches are released

## What is Auto-merge in Dependabot?
Auto-merge is a setting that allows PRs including those created by Dependabot to be automatically merged into the main branch once certain conditions are met (like passing tests or getting approvals).
Example:
- Dependabot detects a new version of lodash.
- It creates a PR to update package.json.
- Tests pass → the PR gets auto-merged without human review.

## Than what is the Misconfiguration?
A Dependabot Auto-merge Misconfiguration happens when:
- Auto-merge is enabled without strict controls (like branch protection, required reviews, or trusted sources)
- An attacker injects or poisons a dependency, and the workflow automatically merges the malicious update
- No human ever reviews the code and it lands directly into main
- It’s not about using Dependabot or auto-merge itself. It’s about enabling auto-merge for dependency updates without applying any guardrails, like:
	- Review requirements
	- Branch protection
	- Source verification
	- Trust boundaries

## The Core Problem:
You’re letting a machine (Dependabot) pull in and merge code into your main branch possibly from external, attacker-controlled sources without any human oversight.

