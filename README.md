# Flux CD GitOps Image Automation with GitHub Actions and Docker Hub

## 1. Project Overview

This project demonstrates how to implement an automated GitOps-based CI/CD workflow using **GitHub Actions, Docker, Docker Hub, Flux CD, and Kubernetes**.

The main objective is to automate the application build, Docker image creation, image publishing, image tag detection, Git repository updates, and Kubernetes deployment updates.

GitHub Actions builds and pushes a new Docker image whenever application source files change. Flux CD continuously monitors Docker Hub for new image tags, applies the configured image selection policy, and updates the Kubernetes deployment manifest through Git.

Flux CD then reconciles the updated configuration and deploys the selected image to the Kubernetes cluster.

The entire workflow follows the GitOps approach, where Git acts as the source of truth for the desired application state.

## 2. Project Objectives

* Automate Docker image builds using GitHub Actions.
* Push Docker images to Docker Hub.
* Automatically discover available Docker image tags using Flux CD.
* Select an image tag using Flux ImagePolicy.
* Automatically update Kubernetes manifests using Flux ImageUpdateAutomation.
* Commit image tag changes to the Git repository.
* Deploy updated application versions through Flux CD reconciliation.
* Reduce manual intervention in the application deployment process.
* Demonstrate an end-to-end GitOps image automation workflow.

## 3. Technologies Used

| Technology     | Purpose                       |
| -------------- | ----------------------------- |
| Git            | Version control               |
| GitHub         | Source code repository        |
| GitHub Actions | CI pipeline automation        |
| Docker         | Container image creation      |
| Docker Hub     | Container image registry      |
| Flux CD        | GitOps continuous delivery    |
| Kubernetes     | Container orchestration       |
| kubectl        | Kubernetes cluster management |
| PowerShell     | Local command execution       |

## 4. Project Architecture

```text
                 Developer
                     |
                     | Push application changes
                     v
             GitHub Repository
                     |
                     v
             GitHub Actions
                     |
                     | Build Docker image
                     | Generate image tag
                     v
                Docker Hub
                     |
                     | Publish new image
                     |
                     v
          Flux ImageRepository
                     |
                     | Scan available image tags
                     v
             Flux ImagePolicy
                     |
                     | Select matching tag
                     v
        Flux ImageUpdateAutomation
                     |
                     | Update Kubernetes manifest
                     | Commit changes to Git
                     v
             GitHub Repository
                     |
                     | Reconcile Git source
                     v
               Flux CD Kustomization
                     |
                     | Apply desired configuration
                     v
              Kubernetes Cluster
                     |
                     v
             Updated Application
```

### Workflow Explanation

**Step 1: Developer pushes code**

The developer pushes application changes to the `main` branch.

**Step 2: GitHub Actions starts**

The GitHub Actions workflow builds a Docker image from the application directory.

**Step 3: Image is published**

The workflow authenticates with Docker Hub and pushes the newly built image using a commit-based tag.

**Step 4: Flux scans image tags**

Flux ImageRepository periodically scans the Docker Hub repository for available image tags.

**Step 5: Flux selects an image**

Flux ImagePolicy evaluates the discovered tags and selects a tag matching the configured filtering and selection rules.

**Step 6: Flux updates Git**

Flux ImageUpdateAutomation updates the image reference in the Kubernetes deployment manifest and commits the change to the configured Git branch.

**Step 7: Flux deploys the application**

Flux CD reconciles the updated Git configuration and applies the desired deployment state to Kubernetes.

**Step 8: Kubernetes updates the application**

Kubernetes performs a rolling update when the deployment's pod template changes and the new image can be pulled successfully.

---

## 5. Repository Structure

```text
fluxcd-gitops-image-automation/
│
├── .github/
│   └── workflows/
│       └── build.yml
│
├── app/
│   └── base/
│       ├── deployment.yaml
│       └── kustomization.yaml
│       └── service.yaml
│       └── index.html
│       └── Dockerfile
│
├── clusters/
│   └── minikube/
│       ├── flux-system
│           ├── kustomization.yaml
│           └── gotk-sync.yaml
│           └── gotk-components.yaml
│       │
│       └── image-automation/
│           
│           ├── image-repository.yaml
│           ├── image-policy.yaml
│           └── image-update-automation.yaml
│
└── README.md
```

### Directory Explanation

**`.github/workflows/build.yml`**

Contains the GitHub Actions CI workflow responsible for building and publishing Docker images.

**`app/base/deployment.yaml`**

Defines the Kubernetes Deployment and its container image.

**`app/base/kustomization.yaml`**

Defines the Kubernetes resources managed by Kustomize for the application.

**`clusters/minikube/kustomization.yaml`**

Defines the resources that Flux CD applies for the Minikube environment.

**`clusters/minikube/image-automation/image-repository.yaml`**

Defines the Docker image repository Flux scans.

**`clusters/minikube/image-automation/image-policy.yaml`**

Defines the filtering and selection rules for Docker image tags.

**`clusters/minikube/image-automation/image-update-automation.yaml`**

Defines how Flux updates image references in Git and commits the changes.

---

## 6. Docker Hub Configuration

The Docker image used in this project is:

```text
docker.io/boobesh2005/fluxcd-gitops-image-automation
```

The repository contains application images with commit-based tags.

Example:

```text
docker.io/boobesh2005/fluxcd-gitops-image-automation:b40824d
docker.io/boobesh2005/fluxcd-gitops-image-automation:b6c9ee6
docker.io/boobesh2005/fluxcd-gitops-image-automation:fe040a6
```

These tags are examples of previously observed image tags. New tags depend on the commits and workflow runs.

Using commit-based tags helps identify which source revision produced an image.

**Important:** The image tags must be unique and must match the format accepted by the Flux ImagePolicy.

---

## 7. GitHub Actions CI Workflow

File:

```text
.github/workflows/build.yml
```

The workflow automates the following tasks:

1. Detect changes to application source files.
2. Check out the Git repository.
3. Set up Docker Buildx.
4. Authenticate with Docker Hub.
5. Generate an image tag from the Git commit SHA.
6. Build the Docker image.
7. Push the image to Docker Hub.

The workflow runs on pushes to `main` when relevant application files change, excluding `app/base/deployment.yaml`, and also supports manual execution through `workflow_dispatch`.

### Required GitHub Secrets

Configure the following secrets in the repository's GitHub Actions secrets settings:

| Secret               | Purpose                 |
| -------------------- | ----------------------- |
| `DOCKERHUB_USERNAME` | Docker Hub username     |
| `DOCKERHUB_TOKEN`    | Docker Hub access token |

Do not hardcode Docker Hub credentials in the workflow file.

### Workflow Example

The following is a reference implementation of the image build and push workflow. If your existing workflow includes additional manifest-update or commit steps, retain those steps when updating the file.

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main
    paths:
      - "app/**"
      - "!app/base/deployment.yaml"
      - ".github/workflows/build.yml"

  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    name: Build and Push Image
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set Up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Generate Image Metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: docker.io/boobesh2005/fluxcd-gitops-image-automation
          tags: |
            type=sha,format=short

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v6
        with:
          context: ./app/base
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

### How the Workflow Works

**Checkout Repository**

Downloads the repository contents into the GitHub Actions runner.

**Set Up Docker Buildx**

Prepares Docker Buildx for building container images.

**Generate Image Metadata**

Generates a short commit SHA tag for the image.

**Login to Docker Hub**

Authenticates using the configured GitHub Actions secrets.

**Build and Push Docker Image**

Builds the image using the Docker build context in `app/base` and pushes it to Docker Hub.

### Important Workflow Notes

* The build context must contain a valid Dockerfile.
* The Docker Hub repository must exist and be accessible.
* The generated tag must satisfy the Flux ImagePolicy filter.
* The workflow should not rebuild images because Flux updates only the deployment manifest.
* If the workflow commits changes to Git, use appropriate permissions and avoid creating a circular workflow trigger.

The example above builds and pushes the image. It does not itself commit the updated Kubernetes manifest; that operation is handled by Flux ImageUpdateAutomation in this project.

---

## 8. Flux CD GitOps Configuration

Flux CD is configured to monitor the Git repository:

```text
ssh://git@github.com/boobeshwaranhati/fluxcd-gitops-image-automation.git
```

Branch:

```text
main
```

The main Flux resources are configured in the `flux-system` namespace.

### 8.1 ImageRepository

File:

```text
clusters/minikube/image-automation/image-repository.yaml
```

Resource name:

```text
demo-image
```

Namespace:

```text
flux-system
```

The ImageRepository points to the Docker Hub image and scans it periodically.

Example configuration:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1
kind: ImageRepository
metadata:
  name: demo-image
  namespace: flux-system
spec:
  image: docker.io/boobesh2005/fluxcd-gitops-image-automation
  interval: 1m
  exclusionList:
    - '^.*\.sig$'
```

### Explanation

| Field                | Purpose                                                        |
| -------------------- | -------------------------------------------------------------- |
| `metadata.name`      | Identifies the image repository resource                       |
| `metadata.namespace` | Defines the namespace containing the resource                  |
| `spec.image`         | Specifies the Docker image repository                          |
| `spec.interval`      | Configures how frequently Flux scans for image tags            |
| `spec.exclusionList` | Excludes image tags matching the configured regular expression |

The one-minute interval means Flux periodically checks for image changes. It does not guarantee that the complete deployment finishes within one minute.

---

### 8.2 ImagePolicy

File:

```text
clusters/minikube/image-automation/image-policy.yaml
```

Resource name:

```text
demo-policy
```

The ImagePolicy references `demo-image` and filters tags using the expression:

```text
^[a-f0-9]{7}$
```

This expression matches a seven-character lowercase hexadecimal tag, such as:

```text
b40824d
b6c9ee6
fe040a6
```

Example configuration:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1
kind: ImagePolicy
metadata:
  name: demo-policy
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: demo-image
  filterTags:
    pattern: '^[a-f0-9]{7}$'
    extract: '$0'
  policy:
    alphabetical:
      order: asc
```

### Explanation

**`imageRepositoryRef`**

Connects the policy to the `demo-image` resource.

**`filterTags`**

Allows only tags matching the configured seven-character hexadecimal pattern.

**`policy.alphabetical`**

Selects a tag according to alphabetical ordering.

**Important:** Alphabetical ordering does not necessarily select the newest image. A lexicographically higher SHA tag is not guaranteed to represent a later commit. If your requirement is to deploy the newest build reliably, consider a tag strategy that encodes an ordered build number or another suitable version identifier, and configure the ImagePolicy accordingly.

---

### 8.3 ImageUpdateAutomation

File:

```text
clusters/minikube/image-automation/image-update-automation.yaml
```

Resource name:

```text
fluxcd-demo-automation
```

This resource updates image references in Git when the selected image tag changes.

The configuration uses:

* Git repository: `flux-system`
* Branch: `main`
* Update path: `./clusters/minikube`
* Strategy: `Setters`
* Reconciliation interval: `1m`

Example configuration:

```yaml
apiVersion: image.toolkit.fluxcd.io/v1
kind: ImageUpdateAutomation
metadata:
  name: fluxcd-demo-automation
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: flux-system
  git:
    checkout:
      ref:
        branch: main
    push:
      branch: main
    commit:
      author:
        name: Flux Image Automation
        email: flux-image-automation@example.com
      messageTemplate: "Update image tags"
  update:
    path: ./clusters/minikube
    strategy: Setters
```

### Explanation

**`interval`**

Defines how frequently Flux checks whether image updates are required.

**`sourceRef`**

Identifies the GitRepository resource used by the automation.

**`checkout.ref.branch`**

Defines the branch Flux checks out before applying updates.

**`push.branch`**

Defines the branch to which Flux pushes image tag changes.

**`commit.author`**

Defines the author information for automated commits.

**`messageTemplate`**

Defines the commit message.

**`update.path`**

Limits manifest updates to the specified directory.

**`strategy: Setters`**

Uses image policy setter markers in the manifests to identify image fields that Flux should update.

---

## 9. Kubernetes Deployment and Image Setter

File:

```text
app/base/deployment.yaml
```

The deployment references the application image and includes an image policy setter marker.

Example image reference:

```yaml
image: boobesh2005/fluxcd-gitops-image-automation:b40824d # {"$imagepolicy": "flux-system:demo-policy"}
```

The marker:

```text
{"$imagepolicy": "flux-system:demo-policy"}
```

connects the image field to the `demo-policy` ImagePolicy in the `flux-system` namespace.

When Flux ImageUpdateAutomation detects a different selected tag, it can update this image reference in Git.

### Why the Setter Marker Matters

Without a correctly configured setter marker, Flux may discover image tags but fail to update the intended image field.

The marker must identify the correct namespace and ImagePolicy name.

Also ensure that the image field is included within the directory configured by `spec.update.path`.

---

## 10. Flux Kustomization Configuration

The Flux Kustomization named `flux-system` reconciles the configured Git source.

Check its status using:

```powershell
flux get kustomizations -A
```

Inspect its details:

```powershell
kubectl describe kustomization flux-system -n flux-system
```

The cluster-level Kustomize configuration must include the application resources and the image automation resources in the correct dependency order.

For example, the image automation directory should contain a `kustomization.yaml` that includes:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - image-repository.yaml
  - image-policy.yaml
  - image-update-automation.yaml
```

Ensure that the parent configuration references this directory. The file existing in Git does not automatically mean Flux is reconciling it.

---

## 11. Complete End-to-End Workflow

### Step 1: Make an Application Change

Open the repository:

```powershell
cd C:\Users\boobe\fluxcd-gitops-image-automation
```

Modify an application source file inside `app/`, such as the HTML page or another application file used by your Docker build.

### Step 2: Commit and Push the Change

```powershell
git add app/
git commit -m "Update application"
git push origin main
```

The GitHub Actions workflow should start if the changed files match its trigger conditions.

### Step 3: Verify GitHub Actions

Open the repository on GitHub.

Navigate to:

```text
Actions → Build and Push Docker Image
```

Verify that the workflow completes successfully.

### Step 4: Verify the Docker Image

Open the Docker Hub repository:

```text
https://hub.docker.com/r/boobesh2005/fluxcd-gitops-image-automation
```

Check that the new image tag is available.

### Step 5: Reconcile the ImageRepository

Run:

```powershell
flux reconcile image repository demo-image -n flux-system
```

This requests an immediate reconciliation of the image repository.

### Step 6: Check Available Image Tags

```powershell
flux get image repository -A
```

Inspect the ImageRepository:

```powershell
kubectl describe imagerepository demo-image -n flux-system
```

Check the ImagePolicy:

```powershell
flux get image policy -A
```

Inspect the policy:

```powershell
kubectl describe imagepolicy demo-policy -n flux-system
```

Verify that the newly published tag matches the filter and that the policy selects the expected image.

### Step 7: Reconcile Image Automation

```powershell
flux reconcile image update fluxcd-demo-automation -n flux-system
```

Check its status:

```powershell
flux get image update -A
```

Inspect the resource:

```powershell
kubectl describe imageupdateautomation fluxcd-demo-automation -n flux-system
```

If an update is available, Flux ImageUpdateAutomation should update the image reference and push a commit to the configured Git branch, provided the Git credentials and repository permissions are correct.

### Step 8: Reconcile the Git Source

```powershell
flux reconcile source git flux-system -n flux-system
```

Check the Git source:

```powershell
flux get sources git -A
```

A healthy source should report `READY=True` and show the revision that Flux has fetched.

### Step 9: Reconcile the Kubernetes Configuration

```powershell
flux reconcile kustomization flux-system -n flux-system
```

Check the Kustomization:

```powershell
flux get kustomizations -A
```

### Step 10: Verify the Deployment

Check the application deployment:

```powershell
kubectl get deployments -A
```

Check the pods:

```powershell
kubectl get pods -A
```

Inspect the application deployment in the `default` namespace:

```powershell
kubectl describe deployment fluxcd-demo -n default
```

Check the deployed image:

```powershell
kubectl get deployment fluxcd-demo -n default -o jsonpath="{.spec.template.spec.containers[*].image}"
```

Check rollout status:

```powershell
kubectl rollout status deployment/fluxcd-demo -n default
```

The final image should match the image tag that Flux selected and committed to Git.

---

## 12. Useful Flux CD Commands

### Check Git Sources

```powershell
flux get sources git -A
```

### Check Image Repositories

```powershell
flux get image repository -A
```

### Check Image Policies

```powershell
flux get image policy -A
```

### Check Image Update Automation

```powershell
flux get image update -A
```

### Check Kustomizations

```powershell
flux get kustomizations -A
```

### Reconcile the Image Repository

```powershell
flux reconcile image repository demo-image -n flux-system
```

### Reconcile Image Automation

```powershell
flux reconcile image update fluxcd-demo-automation -n flux-system
```

### Reconcile the Git Source

```powershell
flux reconcile source git flux-system -n flux-system
```

### Reconcile the Kustomization

```powershell
flux reconcile kustomization flux-system -n flux-system
```

### Check Flux Controllers

```powershell
kubectl get pods -n flux-system
```

### Check Flux Events

```powershell
kubectl get events -n flux-system --sort-by=.metadata.creationTimestamp
```

---

## 13. Access the Application Using Port Forwarding

The application container listens on port `80`.

If the application is exposed through a Kubernetes Service named `fluxcd-demo`, run:

```powershell
kubectl port-forward svc/fluxcd-demo 8080:80 -n default
```

Then open:

```text
http://localhost:8080
```

Keep the PowerShell terminal running while using port forwarding.

If the Service has a different name, replace `fluxcd-demo` with the actual Service name.

To check available Services:

```powershell
kubectl get svc -n default
```

If no Service exists, port-forward the Deployment directly:

```powershell
kubectl port-forward deployment/fluxcd-demo 8080:80 -n default
```

---

## 14. Troubleshooting

### Problem 1: GitHub Actions Workflow Does Not Start

**Possible causes:**

* Changes were made outside the configured trigger paths.
* The workflow file contains an invalid configuration.
* The workflow was not pushed to the expected branch.

**Check:**

```powershell
git status
git branch
git log -1 --oneline
```

Open the GitHub Actions page and inspect the workflow triggers.

### Problem 2: Docker Image Is Not Published

**Possible causes:**

* Incorrect Docker Hub credentials.
* Missing GitHub Actions secrets.
* An invalid Docker build context.
* A missing Dockerfile.

**Check:**

```powershell
docker build -t fluxcd-demo ./app/base
```

This command tests whether the image can be built locally.

### Problem 3: Flux Cannot Discover the Image Tag

**Possible causes:**

* The image repository name is incorrect.
* Docker Hub access is restricted.
* The tag does not match the ImagePolicy filter.
* Flux has not completed a successful scan.

**Check:**

```powershell
kubectl describe imagerepository demo-image -n flux-system
```

```powershell
kubectl describe imagepolicy demo-policy -n flux-system
```

Confirm that the image tag matches:

```text
^[a-f0-9]{7}$
```

### Problem 4: ImageUpdateAutomation Does Not Commit Changes

**Possible causes:**

* The selected tag has not changed.
* The image setter marker is missing or incorrect.
* The update path does not contain the deployment manifest.
* Git push permissions are insufficient.
* The GitRepository configuration or authentication is incorrect.

**Check:**

```powershell
kubectl describe imageupdateautomation fluxcd-demo-automation -n flux-system
```

Verify the `Ready` condition and the latest reconciliation message.

### Problem 5: Git Source Does Not Fetch the New Commit

**Check:**

```powershell
flux reconcile source git flux-system -n flux-system
```

```powershell
flux get sources git -A
```

Confirm that the Git source reports a ready status and the expected revision.

### Problem 6: Kubernetes Does Not Deploy the New Image

**Check the Deployment:**

```powershell
kubectl describe deployment fluxcd-demo -n default
```

**Check the Pods:**

```powershell
kubectl get pods -n default
```

**Check Pod Events:**

```powershell
kubectl get events -n default --sort-by=.metadata.creationTimestamp
```

**Check the Current Image:**

```powershell
kubectl get deployment fluxcd-demo -n default -o jsonpath="{.spec.template.spec.containers[*].image}"
```

Remember that publishing an image alone does not change the running deployment. The selected tag must be committed to the manifest and reconciled by Flux.

### Problem 7: GitOps Reconciliation Reverts a Manual Change

This is expected GitOps behavior when a manual Kubernetes change conflicts with the desired configuration stored in Git.

Make the intended configuration change in the Git repository instead of relying on manual edits to the running resource.

---

## 15. Security Considerations

* Store Docker Hub credentials in GitHub Actions secrets.
* Never commit passwords, access tokens, or private keys to Git.
* Use a dedicated Docker Hub access token with the required permissions.
* Ensure Flux has appropriate Git repository access.
* Avoid granting unnecessary repository write permissions.
* Keep credentials out of logs and documentation.
* Review automated Git commits before using the workflow in a production environment.

---

## 16. Expected Results

After successful execution, the project should demonstrate the following:

* A source code change triggers GitHub Actions.
* GitHub Actions builds and publishes a Docker image.
* Docker Hub contains the newly generated image tag.
* Flux ImageRepository discovers available image tags.
* Flux ImagePolicy selects a tag according to its configured policy.
* Flux ImageUpdateAutomation updates the deployment manifest in Git.
* Flux commits the updated manifest to the configured branch.
* Flux CD reconciles the updated Git configuration.
* Kubernetes deploys the selected image.
* The application can be accessed using the configured Service or port-forwarding.

---

## 17. Key Learnings

Through this project, I gained practical experience with:

* Implementing GitHub Actions CI workflows.
* Building and publishing Docker images.
* Managing container images through Docker Hub.
* Implementing GitOps with Flux CD.
* Configuring Flux ImageRepository and ImagePolicy.
* Automating image tag updates using ImageUpdateAutomation.
* Managing Kubernetes deployment manifests using Git.
* Understanding Git source reconciliation and Kustomization reconciliation.

---

## 18. Future Improvements

Potential improvements include:

* Using a versioning strategy that reliably identifies the latest build.
* Adding image vulnerability scanning.
* Adding automated tests before publishing images.
* Using separate development, QA, and production environments.
* Implementing deployment health checks and rollback procedures.
* Adding notifications for failed builds and reconciliation errors.
* Introducing image signature verification and stronger supply-chain security.

---

## 19. Conclusion

This project demonstrates an automated GitOps workflow using GitHub Actions, Docker Hub, Flux CD, and Kubernetes.

GitHub Actions automates the image build and publishing process, while Flux CD discovers image tags, updates Kubernetes manifests in Git, and reconciles the desired application state.

The result is a workflow that reduces manual deployment steps and keeps the Kubernetes application aligned with the configuration stored in Git.

**Project:** Flux CD GitOps Image Automation

**Container Registry:** Docker Hub

**CI Tool:** GitHub Actions

**GitOps Tool:** Flux CD

**Orchestration:** Kubernetes
