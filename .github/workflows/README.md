# GitHub Actions CI/CD Pipeline

This directory contains the GitHub Actions workflow for the Tic Tac Toe application's CI/CD pipeline.

## Pipeline Stages

The CI/CD pipeline consists of the following stages:

1. **Unit Testing** - Runs the test suite using Vitest
2. **Static Code Analysis** - Performs linting with ESLint
3. **Build** - Creates a production build of the application
4. **Docker Image Creation** - Builds a Docker image using a multi-stage Dockerfile
5. **Docker Image Scan** - Scans the image for vulnerabilities using Trivy
6. **Docker Image Push** - Pushes the image to GitHub Container Registry
7. **Update Kubernetes Deployment** - Updates the Kubernetes deployment file with the new image tag

## Repository Name Considerations

Docker images require lowercase repository names. If your GitHub repository contains uppercase characters (e.g., `Wandati/devsecops-demo` instead of `wandati/devsecops-demo`), the pipeline will fail when:

1. Building and pushing Docker images
2. Running vulnerability scans with Trivy
3. Updating Kubernetes deployment files

### Handling Repository Names with Uppercase Characters

If your repository name includes uppercase characters, you need to modify the workflow file by:

1. Adding a step to convert the repository name to lowercase before using it for Docker operations:

```yaml
- name: Set lowercase repository name
  id: repo_name
  run: echo "REPO_NAME_LOWER=$(echo ${{ github.repository }} | tr '[:upper:]' '[:lower:]')" >> $GITHUB_ENV
```

2. Using the lowercase name in all image references:

```yaml
# In Docker metadata action
images: ${{ env.REGISTRY }}/${{ env.REPO_NAME_LOWER }}

# In Trivy scanner
image-ref: ${{ env.REGISTRY }}/${{ env.REPO_NAME_LOWER }}:sha-${{ github.sha }}

# In Kubernetes deployment update
NEW_IMAGE="${REGISTRY}/${REPO_NAME_LOWER}:${IMAGE_TAG}"
```

### For Repositories with Lowercase Names

If your repository name is already lowercase (e.g., `wandati/devsecops-demo`), you can use the standard workflow without modifications.

## How the Kubernetes Deployment Update Works

The "Update Kubernetes Deployment" stage:

1. Runs only on pushes to the main branch
2. Uses a shell script to update the image reference in the Kubernetes deployment file
3. Commits and pushes the updated deployment file back to the repository
4. This ensures that the Kubernetes manifest always references the latest image

## Required Secrets

The workflow requires the following GitHub secrets:

- `GITHUB_TOKEN` - Automatically provided by GitHub Actions, used for pushing to the repository
- `TOKEN` - Personal access token with appropriate permissions to push to the container registry and repository

## Continuous Deployment

For full continuous deployment, you would need to:

1. Set up a Kubernetes operator like Flux or ArgoCD to watch for changes in the repository
2. Configure it to automatically apply changes to the Kubernetes manifests
3. This would complete the CI/CD pipeline by automatically deploying the new image to your Kubernetes cluster

## Manual Deployment

If you're not using a GitOps approach with an operator, you can manually apply the updated deployment:

```bash
kubectl apply -f kubernetes/deployment.yaml
```

Or set up a webhook to trigger the deployment when the manifest is updated.

## Troubleshooting

Common issues and solutions:

1. **Failed Docker image builds or pushes due to case sensitivity**: Follow the repository name conversion steps above.
2. **Trivy scan failures**: Ensure the image name is properly lowercase in the Trivy action.
3. **ArgoCD/Flux deployment issues**: Make sure the image name in your Kubernetes manifests matches the lowercase repository name format.