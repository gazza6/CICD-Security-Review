### Malicious Container Images in Pipelines (Public Docker Hub)
- Many CI/CD pipelines pull container images from public registries (like Docker Hub) for building, testing, and deploying applications
- If these images are not verified or pinned by digest, attackers can:
    - Upload malicious images with backdoors, cryptominers, or credential stealers
    - Replace legitimate images with poisoned ones (if the image name is reused or namespace isn’t protected)
    - Abuse popular image names (typosquatting or dependency confusion).

