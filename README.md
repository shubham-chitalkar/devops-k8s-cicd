# DevOps Kubernetes CI/CD Project

A complete DevOps CI/CD project that demonstrates how to automatically build, version, publish, and deploy a containerized Python application to Kubernetes using **GitHub Actions, Docker Hub, a self-hosted GitHub Actions runner, and Minikube**.

---

## 📌 Project Overview

This project implements an end-to-end CI/CD pipeline:

```text
Developer
    │
    │ git push
    ▼
GitHub Repository
    │
    ▼
GitHub Actions - Docker CI
    │
    ├── Checkout code
    ├── Build Docker image
    ├── Tag image with Git commit SHA
    └── Push image to Docker Hub
    │
    ▼
Docker Hub
    │
    ▼
GitHub Actions - Kubernetes CD
    │
    ▼
Self-Hosted GitHub Actions Runner
    │
    ▼
kubectl
    │
    ▼
Minikube Kubernetes Cluster
    │
    ├── Deployment
    │      └── 2 Pods
    │
    ├── Service
    │
    └── Ingress
    │
    ▼
Application
```

---

# 🛠️ Technologies Used

* Git
* GitHub
* GitHub Actions
* Docker
* Docker Hub
* Python
* Kubernetes
* kubectl
* Minikube
* NGINX Ingress Controller
* GitHub Actions Self-Hosted Runner
* Linux / Ubuntu

---

# 📁 Project Structure

```text
devops-k8s-cicd/
│
├── app/
│   ├── Dockerfile
│   ├── app.py
│   ├── requirements.txt
│   └── .dockerignore
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
│
├── .github/
│   └── workflows/
│       ├── docker-ci.yml
│       └── cd.yml
│
└── .gitignore
```

---

# 1. Application

The application is a simple Python application.

The application exposes port:

```text
5000
```

Example response:

```text
Hello DevOps! Version 1.1
```

The purpose of the application is not complexity. It is used to demonstrate the complete DevOps deployment pipeline.

---

# 2. Docker

## Dockerfile

The application uses:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

### Docker process

The Docker image:

1. Uses Python 3.12 slim.
2. Creates `/app` as the working directory.
3. Copies `requirements.txt`.
4. Installs Python dependencies.
5. Copies the application.
6. Exposes port `5000`.
7. Starts the Python application.

---

# 3. Docker Ignore

`app/.dockerignore`:

```text
venv/
__pycache__/
.git/
```

This prevents unnecessary files from being included in the Docker build context.

---

# 4. Git Ignore

`.gitignore`:

```text
venv/
__pycache__/
.env
```

This prevents the local Python virtual environment, Python cache files, and environment files from being committed to Git.

---

# 5. Docker CI Pipeline

File:

```text
.github/workflows/docker-ci.yml
```

The CI workflow runs when code is pushed to the `main` branch.

```yaml
name: Docker CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker image
        run: docker build -t shubham336/devops-k8s-app:${{ github.sha }} ./app

      - name: Push Docker image
        run: docker push shubham336/devops-k8s-app:${{ github.sha }}
```

## What CI does

When we run:

```bash
git push
```

GitHub Actions automatically:

```text
Git Push
   ↓
Checkout
   ↓
Docker Login
   ↓
Docker Build
   ↓
Tag with Git SHA
   ↓
Push to Docker Hub
```

---

# 6. Why Git Commit SHA Is Used as the Docker Tag

Instead of using only:

```text
latest
```

the pipeline uses:

```text
${{ github.sha }}
```

Example:

```text
shubham336/devops-k8s-app:495ddc4a773db533a128786f130f8a23c58a8465
```

This gives every build a unique version.

Benefits:

* Easy version tracking
* Easy rollback
* Exact connection between Git commit and Docker image
* Avoids ambiguity caused by the `latest` tag

---

# 7. Docker Hub

Docker images are pushed to:

```text
shubham336/devops-k8s-app
```

Example:

```text
shubham336/devops-k8s-app:<commit-sha>
```

Docker Hub authentication is handled using GitHub Secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

Secrets are not stored directly inside the workflow.

---

# 8. Kubernetes Deployment

File:

```text
k8s/deployment.yaml
```

The application is deployed using a Kubernetes Deployment.

Important configuration:

```yaml
replicas: 2
```

The Deployment maintains two application Pods.

Example:

```text
Deployment
    │
    ├── Pod 1
    │
    └── Pod 2
```

The image is versioned using the Git commit SHA.

Example:

```text
shubham336/devops-k8s-app:<commit-sha>
```

---

# 9. Kubernetes Service

File:

```text
k8s/service.yaml
```

The Service is:

```text
ClusterIP
```

Configuration:

```text
port: 80
targetPort: 5000
```

This means:

```text
Service port 80
      ↓
Pod port 5000
```

The Service selects Pods using:

```yaml
selector:
  app: devops-app
```

We verified that the Service had two healthy endpoints.

---

# 10. Kubernetes Ingress

File:

```text
k8s/ingress.yaml
```

The Ingress uses:

```text
ingressClassName: nginx
```

Traffic flow:

```text
Client
  ↓
NGINX Ingress
  ↓
devops-app-service:80
  ↓
Pods:5000
```

The Ingress routes `/` to:

```text
devops-app-service
```

---

# 11. Minikube

The Kubernetes cluster used during development is Minikube.

Check Minikube:

```bash
minikube status
```

Example:

```text
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Check Kubernetes context:

```bash
kubectl config current-context
```

Expected:

```text
minikube
```

---

# 12. Kubernetes Verification

Check Deployment:

```bash
kubectl get deployment devops-app
```

Expected:

```text
NAME         READY   UP-TO-DATE   AVAILABLE
devops-app   2/2     2            2
```

Check Pods:

```bash
kubectl get pods
```

Expected:

```text
devops-app-xxxxx   1/1   Running
devops-app-xxxxx   1/1   Running
```

Check Service:

```bash
kubectl get service devops-app-service
```

Check Ingress:

```bash
kubectl get ingress
```

---

# 13. Kubernetes Service Testing

We tested the application from inside the Kubernetes cluster using a temporary curl Pod:

```bash
kubectl run curl-test --rm -it --restart=Never \
  --image=curlimages/curl \
  -- curl -s http://devops-app-service
```

Expected output:

```text
Hello DevOps! Version 1.1
```

This confirmed:

```text
Pod
 ↓
Service
 ↓
Application
```

was working correctly.

---

# 14. Rolling Update

When a new Docker image is deployed, Kubernetes performs a rolling update.

We used:

```bash
kubectl set image deployment/devops-app \
  devops-app=shubham336/devops-k8s-app:<commit-sha>
```

Then verified:

```bash
kubectl rollout status deployment/devops-app
```

Expected:

```text
deployment "devops-app" successfully rolled out
```

We also verified the image:

```bash
kubectl get deployment devops-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

---

# 15. Kubernetes Rollout History

We checked Deployment history using:

```bash
kubectl rollout history deployment/devops-app
```

Example:

```text
REVISION
1
2
3
```

Specific revision:

```bash
kubectl rollout history deployment/devops-app --revision=3
```

This demonstrated Kubernetes Deployment revision tracking.

---

# 16. ReplicaSet

We also verified ReplicaSets:

```bash
kubectl get rs
```

During a rolling update, Kubernetes creates a new ReplicaSet and scales down the old ReplicaSet.

Example:

```text
Old ReplicaSet
    ↓
0 Pods

New ReplicaSet
    ↓
2 Pods
```

This demonstrates how Kubernetes manages Deployment updates.

---

# 17. The Main CD Problem We Solved

Initially, our CD workflow used:

```yaml
runs-on: ubuntu-latest
```

The problem was:

```text
GitHub-hosted runner
        ↓
       ❌
Local Minikube
```

The GitHub-hosted runner runs on GitHub infrastructure, while Minikube runs locally on our machine.

Therefore, the GitHub runner could not directly access our local Kubernetes cluster.

---

# 18. Solution — Self-Hosted GitHub Actions Runner

We installed a GitHub Actions self-hosted runner on the same machine where Minikube is running.

Architecture:

```text
GitHub
   ↓
Self-Hosted Runner
   ↓
Local Machine
   ↓
kubectl
   ↓
Minikube
   ↓
Kubernetes
```

The runner was registered with the labels:

```text
self-hosted
Linux
X64
```

The runner was started using:

```bash
cd ~/actions-runner
./run.sh
```

Successful connection:

```text
Connected to GitHub
Listening for Jobs
```

---

# 19. Kubernetes CD Pipeline

File:

```text
.github/workflows/cd.yml
```

The CD workflow is triggered after the Docker CI workflow completes.

```yaml
name: Kubernetes CD

on:
  workflow_run:
    workflows: ["Docker CI"]
    types:
      - completed

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set image tag
        run: echo "IMAGE_TAG=${{ github.event.workflow_run.head_sha }}" >> $GITHUB_ENV

      - name: Check kubectl
        run: |
          kubectl version --client
          kubectl config current-context || true

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/devops-app \
            devops-app=shubham336/devops-k8s-app:${IMAGE_TAG}

      - name: Check rollout status
        run: |
          kubectl rollout status deployment/devops-app
```

---

# 20. How Automatic CD Works

The complete process is:

```text
1. Developer changes application
        ↓
2. git add
        ↓
3. git commit
        ↓
4. git push
        ↓
5. Docker CI starts
        ↓
6. Docker image is built
        ↓
7. Image receives Git SHA tag
        ↓
8. Image is pushed to Docker Hub
        ↓
9. Docker CI completes
        ↓
10. Kubernetes CD starts
        ↓
11. Self-hosted runner receives job
        ↓
12. IMAGE_TAG is taken from commit SHA
        ↓
13. kubectl updates Deployment
        ↓
14. Kubernetes performs rolling update
        ↓
15. Rollout status is checked
        ↓
16. New Pods become Running
```

---

# 21. Final Working Result

We successfully reached:

```text
Deployment:
2/2 Ready

Pods:
2/2 Running

Docker Image:
shubham336/devops-k8s-app:<Git-SHA>

CD:
Succeeded

Application:
Hello DevOps! Version 1.1
```

The final CD job showed:

```text
Running job: deploy
Job deploy completed with result: Succeeded
```

---

# 22. How to Stop the Project

The project does not need to run 24/7.

## Stop the GitHub Actions Runner

In the terminal running:

```bash
./run.sh
```

press:

```text
Ctrl + C
```

This stops the runner process.

It does not remove the runner registration from GitHub.

---

## Stop Minikube

When finished practicing:

```bash
minikube stop
```

This stops the Minikube VM/cluster while preserving its configuration.

---

# 23. How to Start the Project Again

Before a demonstration, start Minikube:

```bash
minikube start
```

Check:

```bash
minikube status
```

Then:

```bash
kubectl config current-context
```

Expected:

```text
minikube
```

Check the application:

```bash
kubectl get deployment
kubectl get pods
kubectl get service
kubectl get ingress
```

---

# 24. Start the Self-Hosted Runner

Open another terminal:

```bash
cd ~/actions-runner
./run.sh
```

Wait for:

```text
Connected to GitHub
Listening for Jobs
```

Keep this terminal open during the demonstration.

---

# 25. Interview Demonstration

For an interview demo, start:

### Terminal 1 — Kubernetes

```bash
minikube start
```

Then:

```bash
kubectl get pods
```

### Terminal 2 — GitHub Runner

```bash
cd ~/actions-runner
./run.sh
```

Keep it running.

### Terminal 3 — Project

```bash
cd ~/devops-k8s-cicd
```

Show the project structure:

```bash
tree -L 3
```

Show the CI workflow:

```bash
cat .github/workflows/docker-ci.yml
```

Show the CD workflow:

```bash
cat .github/workflows/cd.yml
```

Then make a small application change, for example:

```text
Hello DevOps! Version 1.2
```

Commit and push:

```bash
git add .
git commit -m "Update application version"
git push
```

Then demonstrate:

```text
Git Push
    ↓
Docker CI
    ↓
Docker Build
    ↓
Docker Hub
    ↓
Kubernetes CD
    ↓
Self-Hosted Runner
    ↓
kubectl
    ↓
Minikube
    ↓
Rolling Update
```

Finally verify:

```bash
kubectl get pods
```

and:

```bash
kubectl run curl-test --rm -it --restart=Never \
  --image=curlimages/curl \
  -- curl -s http://devops-app-service
```

The response should show the new application version.

---

# 26. Useful Troubleshooting Commands

## Check Minikube

```bash
minikube status
```

## Check Kubernetes context

```bash
kubectl config current-context
```

## Check all Pods

```bash
kubectl get pods -o wide
```

## Check Deployment

```bash
kubectl get deployment devops-app
```

## Check Deployment image

```bash
kubectl get deployment devops-app \
  -o jsonpath='{.spec.template.spec.containers[0].image}'; echo
```

## Check Service

```bash
kubectl get service devops-app-service
```

## Check Service endpoints

```bash
kubectl get endpoints devops-app-service
```

For newer Kubernetes versions, EndpointSlices can also be checked:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=devops-app-service
```

## Check Ingress

```bash
kubectl get ingress
```

## Check Ingress Controller

```bash
kubectl get pods -n ingress-nginx
```

## Check rollout

```bash
kubectl rollout status deployment/devops-app
```

## Check rollout history

```bash
kubectl rollout history deployment/devops-app
```

## Check ReplicaSets

```bash
kubectl get rs
```

---

# 27. Interview Explanation

A concise explanation of the project:

> "I built an end-to-end CI/CD pipeline for a containerized Python application. Whenever I push code to the main branch, GitHub Actions builds a Docker image, tags it using the Git commit SHA, and pushes it to Docker Hub. After the CI workflow completes, a Kubernetes CD workflow runs on a self-hosted GitHub Actions runner on my local machine. The runner uses kubectl to update the Kubernetes Deployment running on Minikube. Kubernetes performs a rolling update across two replicas, and the rollout status is automatically verified. The application is exposed internally through a ClusterIP Service and configured through an NGINX Ingress."

---

# 28. Key DevOps Concepts Demonstrated

This project demonstrates:

* Git workflow
* GitHub repository management
* GitHub Actions
* CI pipeline
* CD pipeline
* Docker image creation
* Docker image tagging
* Docker Hub registry
* Git SHA-based versioning
* GitHub Secrets
* Self-hosted GitHub Actions runner
* Kubernetes Deployment
* Kubernetes Pods
* ReplicaSets
* Rolling updates
* Kubernetes Service
* ClusterIP
* Kubernetes Ingress
* NGINX Ingress Controller
* kubectl
* Minikube
* Deployment rollout verification
* Basic CI/CD troubleshooting

---

# 29. Final Architecture

```text
                    ┌─────────────────┐
                    │    Developer    │
                    └────────┬────────┘
                             │
                         git push
                             │
                             ▼
                    ┌─────────────────┐
                    │     GitHub      │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Docker CI       │
                    │ GitHub Actions  │
                    └────────┬────────┘
                             │
                       docker build
                             │
                             ▼
                    ┌─────────────────┐
                    │   Docker Hub    │
                    │  SHA-tag image  │
                    └────────┬────────┘
                             │
                    CI completed
                             │
                             ▼
                    ┌─────────────────┐
                    │ Kubernetes CD   │
                    │ GitHub Actions  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Self-Hosted     │
                    │ Runner          │
                    └────────┬────────┘
                             │
                          kubectl
                             │
                             ▼
                    ┌─────────────────┐
                    │    Minikube     │
                    │    Kubernetes   │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
        Deployment        Service        Ingress
          2 Pods          ClusterIP        NGINX
              │              │              │
              └──────────────┴──────────────┘
                             │
                             ▼
                    Python Application
```

---

# ✅ Project Status

**Project: COMPLETE**

```text
Git                         ✅
GitHub                      ✅
Docker                      ✅
Docker Hub                  ✅
GitHub Actions CI           ✅
Self-Hosted Runner          ✅
GitHub Actions CD           ✅
Kubernetes                  ✅
Minikube                    ✅
Deployment                  ✅
Rolling Update              ✅
Service                     ✅
Ingress                     ✅
Application Testing         ✅
End-to-End CI/CD            ✅
```

The project can now be stopped and restarted whenever needed for practice or interview demonstrations.
