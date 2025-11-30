CircleCI is a popular CI/CD platform that integrates with GitHub and other SCM to automate the software development lifecycle.

## CircleCI GitHub Integration
CircleCI integrates with GitHub by connecting to repositories and setting up workflows based on changes.
Key points:
- OAuth Authentication to access GitHub repositories (Pre Sep. 2023)
- Webhooks to trigger builds based on events
- Workflows defined in `.circleci/config.yml`

## CircleCI vs GitHub Actions
- Configuration:
    - CircleCI uses `.circleci/config.yml`
    - GitHub Actions uses `.github/workflows`
- Integration:
    - CircleCI is a separate platform
    - GitHub Actions is native to GitHub
- Interface:
    - CircleCI has a dedicated dashboard
    - GitHub Actions is integrated into GitHub

## CircleCI Personal API Token
- A CircleCI API Token is a bearer token used to authenticate against the CircleCI API (https://circleci.com/api/v2/) 
- It provides programmatic access to CircleCI resources like:
    - Pipelines
    - Projects
    - Environment Variables (Secrets)
    - Contexts (Shared Secrets)
    - Artifacts
    - Organization Details
    - Triggering Builds

How to Identify a CircleCI API Token- CircleCI API tokens have this structure:
- `CCIPA-<random string>`

### How Can These Tokens Can be leaked?
- GitHub/GitLab Repo Leak
- Accidentally committed .circleci/config.yml or .env file containing token
- Build Logs
	- Tokens echoed in build logs or environment variables
- Artifact Theft
	- Downloaded from build artifacts or package outputs
- Internal Recon
	- Stolen from developer laptops or internal documentation
- Phishing
	- Stolen via fake CircleCI login portals or credential theft

