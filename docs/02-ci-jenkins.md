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
