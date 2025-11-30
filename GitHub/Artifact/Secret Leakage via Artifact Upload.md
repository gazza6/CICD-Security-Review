Think of Artifact as "output files" that one stage produces and another stage uses.
## How Artifacts Work in GitHub Actions
- A job generates a file (e.g., build/app.zip)
- That file is uploaded using **actions/upload-artifact**
- Another job or workflow downloads and uses it via **actions/download-artifact**

Example
```yml
- name: Upload results 
  uses: actions/upload-artifact@v3 
  with:
	name: test-logs 
	path: logs/results.json
```

This stores the results.json file so another workflow can fetch and use it.

## How its used in a pipleline
- Avoid redundant processing (e.g., building twice)
- Enable cross-job communication
- Provide traceability (artifacts tied to builds/releases)
- Support manual review or compliance archiving

## Description
Developers or pipelines accidentally include secrets (like API keys, tokens, credentials) in files that get uploaded as artifacts.

Once uploaded:
- These artifacts are publicly accessible if the repo is public.
- Or accessible to any user with pipeline read access in private repos.
- If another workflow or user downloads the artifact → the secret is leaked.

## Common Mistakes
- Debug logs with secrets
  - **env** variables or stack traces get written to logs:
```
export API_KEY=abcd1234 
echo $API_KEY >> logs.txt
```
- Config or .env files included in artifacts
  - **.env, secrets.yaml, or settings.json** gets bundled into an artifact by mistake
- Credentials injected into test reports
  - Test outputs include headers, tokens, or raw responses with secrets
- Uploading entire working directories
  - This often includes **.env, .aws, .npmrc**, or cache directories unintentionally

## Real Attack Scenarios
- An attacker opens a PR, and during testing, secrets get logged and uploaded.
- They download the artifact later from a workflow_run job.
- If permissions are misconfigured, they can use it to access cloud APIs, databases, or even push malicious code back.

## Attack Flow Explanation
1. Attacker watches for workflows that create artifacts
2. A CI job runs after a commit or PR is opened
3. The workflow uploads files, possibly including secrets
4. Attacker retrieves the artifact and extracts sensitive data
5. If possible, attacker abuses token to push malicious changes
6. GITHUB_TOKEN expires, but the secret is already compromised

Mitigation Strategies
- Update to the Latest Version of `upload-artifact`
	- Use the latest stable version of the `upload-artifact` action to avoid any known security vulnerabilities
		- uses: actions/upload-artifact@v4
- Limit Uploaded Files
	- Instead of uploading the entire workspace, specify only the files or folders that are necessary; this prevents sensitive files like `.git` from being exposed
		- path: ./build/logs
- Use Least Privilege for `GITHUB_TOKEN`
  - Modify the default permissions of `GITHUB_TOKEN` in the workflow to limit its scope; by default, `GITHUB_TOKEN` has broad permissions, but you can restrict it to only the permissions needed for the workflow
    - permissions:
    - contents: read
    - actions: none
- Disable Persisting Credentials
  - Disable the default behavior of persisting credentials by setting the `persistcredentials` flag to `false` in `actions/checkout`
```
- uses: actions/checkout@v2 
  with:
	persist-credentials: false
```
- Token Permission Restrictions
  - Always review and set the required permissions for the token in your GitHub
  - A ctions workflow to ensure it follows the principle of least privilege