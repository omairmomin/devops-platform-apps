# Phase 2: CI with Jenkins

## Goal
Set up Jenkins to automatically checkout code from GitHub and build a
Docker image for the `frontend` microservice.

## Jenkins setup
- Jenkins run as a Docker container on the EC2 workstation
  (`jenkins/jenkins:lts` image)
- Docker socket (`/var/run/docker.sock`) mounted into the container so
  Jenkins can build images using the host's Docker daemon
- Docker CLI installed inside the running container (the base Jenkins
  image doesn't ship it) — known limitation, a custom Jenkins image with
  Docker CLI baked in would be the production-grade fix
- GitHub access configured via a Personal Access Token stored as a
  Jenkins credential (`github-credentials`)

## Pipeline (`Jenkinsfile`, stored in the app repo — pipeline as code)
Stages:
1. **Checkout** — pulls the latest code from `devops-platform-apps` (main branch)
2. **Build Docker Image** — builds the `frontend` service image, tagged
   with the Jenkins build number
3. **Verify Image** — confirms the image exists locally

## Access
Jenkins UI reached via SSH port-forward (`-L 8081:localhost:8081`), same
approach as ArgoCD — no ports are exposed publicly on the EC2 security group.

## Result
First pipeline run: SUCCESS. `frontend:2` image built.

## Next
- Add SonarCloud code quality scan
- Add Trivy image vulnerability scan
- Push image to GitHub Container Registry (GHCR)

## Update: Trivy security scan added
- Trivy installed on both the EC2 host and inside the Jenkins container
- Pipeline now scans the built image for HIGH/CRITICAL vulnerabilities
  after build (`--exit-code 0`, non-blocking for now — report-only)
- First scan result: frontend base image (distroless) had 0 OS-level
  vulnerabilities; the Go binary itself showed a few MEDIUM findings,
  no HIGH/CRITICAL
- Can be switched to blocking (`--exit-code 1`) later to fail the build
  on serious vulnerabilities

## Update: Push to GitHub Container Registry (GHCR)
- Added a dedicated GitHub PAT (`write:packages`, `read:packages`) stored as
  the Jenkins credential `ghcr-credentials`, separate from the repo-access token
- Pipeline order: Build -> Trivy scan -> Push, so images are scanned before
  reaching the registry
- Credentials injected with `withCredentials` and passed via
  `--password-stdin`; the token is masked in build logs and never stored in the repo
- Images are tagged with the Jenkins build number
  (`ghcr.io/omairmomin/frontend:<build>`)
- GHCR used instead of ECR to stay within the free budget
