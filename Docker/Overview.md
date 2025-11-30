Why Use Docker in CI/CD?
- Consistency: "Works on my machine" problem solved.
- Scalability: Containers start fast, making pipelines efficient.
- Isolation: Prevents conflicts between dependencies.
- Portability: Runs on any environment (local, cloud, on-premises).

### Docker in a CI/CD Workflow
- Developers push code to a Git repository.
- CI/CD tool (Jenkins, GitHub Actions, GitLab CI, etc.) pulls the code.
- Docker image is built based on a Dockerfile.
- Automated security scans & tests run inside containers.
- Docker image is pushed to a registry (DockerHub, ECR, GCR, etc.).
- CD phase: Deploy to staging/production.

### Key Components of Docker in CI/CD
- Dockerfile: Blueprint for creating Docker images
- Docker Image: Pre-packaged application and dependencies
- Docker Container: Running instance of an image
- Docker Registry: Stores and manages images (DockerHub, AWS ECR, etc.)
- Orchestrators: Kubernetes, Docker Swarm (for scaling and managing deployments)

## Common Docker Security Risks in CI/CD
- Running containers as root (high privilege risks)
- Exposed Docker daemon socket (/var/run/docker.sock exploits)
- Insecure base images (using unverified public images)
- Secrets exposure in Docker images (hardcoded credentials in Dockerfiles)
- Vulnerable third-party dependencies (supply chain risks)