# GCP Build Image Action

A GitHub Action to build and push container images to Google Cloud using Cloud Build with Buildpacks.

## Features

- 🚀 Build container images using Google Cloud Build
- 📦 Uses Cloud Native Buildpacks for automatic image creation
- 🔧 Configurable project, region, and build settings
- ⚡ Simple and lightweight composite action

## Prerequisites

Before using this action, ensure you have:

1. **Google Cloud authentication** set up in your workflow (using `google-github-actions/auth` or similar)
2. **Cloud Build API** enabled in your Google Cloud project
3. **Artifact Registry** (or Container Registry) repository created for storing images
4. **Proper IAM permissions** for the service account:
   - `Cloud Build Editor` or `Cloud Build Service Account`
   - `Artifact Registry Writer` (if using Artifact Registry)

## Usage

### Basic Example

```yaml
name: Build and Deploy
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v1
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
          
      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v1
        
      - name: Build image
        uses: discue/gcp-build-image-action@v1
        with:
          image: 'europe-west3-docker.pkg.dev/my-project/my-repo/my-app'
          project: 'my-project'
          region: 'europe-west3'
```

### Advanced Example

```yaml
      - name: Build image with custom settings
        uses: discue/gcp-build-image-action@v1
        with:
          image: 'europe-west3-docker.pkg.dev/my-project/my-repo/my-app:${{ github.sha }}'
          project: 'my-project'
          region: 'europe-west3'
          source: './src'
          quiet: 'false'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `image` | The full image URL to build (e.g., `europe-west3-docker.pkg.dev/project/repository/image`) | Yes | - |
| `project` | Google Cloud project ID | No | Uses gcloud default |
| `region` | Google Cloud region for builds | No | Uses gcloud default |
| `source` | Source directory to build from | No | `.` |
| `quiet` | Run in quiet mode | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `image-url` | The built image URL |

## Authentication

This action requires Google Cloud authentication. The recommended approach is to use Workload Identity Federation:

```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v1
  with:
    workload_identity_provider: 'projects/123456789/locations/global/workloadIdentityPools/my-pool/providers/my-provider'
    service_account: 'my-service-account@my-project.iam.gserviceaccount.com'
```

Alternatively, you can use a service account key (less secure):

```yaml
- name: Authenticate to Google Cloud
  uses: google-github-actions/auth@v1
  with:
    credentials_json: ${{ secrets.GCP_SA_KEY }}
```

## Required IAM Permissions

The service account used must have the following permissions:

- `cloudbuild.builds.create`
- `cloudbuild.builds.get`
- `artifactregistry.repositories.uploadArtifacts` (if using Artifact Registry)
- `storage.objects.create` (for build logs and cache)

These are typically provided by the following predefined roles:
- `roles/cloudbuild.builds.builder`
- `roles/artifactregistry.writer`

## Example Workflow

```yaml
name: Build and Deploy to Cloud Run

on:
  push:
    branches: [main]

env:
  PROJECT_ID: my-project
  REGION: europe-west3
  REPOSITORY: my-repo
  SERVICE: my-service

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v1
        with:
          workload_identity_provider: ${{ secrets.WIF_PROVIDER }}
          service_account: ${{ secrets.WIF_SERVICE_ACCOUNT }}

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v1

      - name: Build image
        id: build
        uses: discue/gcp-build-image-action@v1
        with:
          image: '${{ env.REGION }}-docker.pkg.dev/${{ env.PROJECT_ID }}/${{ env.REPOSITORY }}/${{ env.SERVICE }}:${{ github.sha }}'
          project: ${{ env.PROJECT_ID }}
          region: ${{ env.REGION }}

      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy ${{ env.SERVICE }} \
            --image ${{ steps.build.outputs.image-url }} \
            --region ${{ env.REGION }} \
            --platform managed \
            --allow-unauthenticated
```

## Error Handling

The action will fail if:
- Google Cloud authentication is not set up
- The specified project or region is invalid
- Cloud Build API is not enabled
- Insufficient IAM permissions
- Invalid image URL format
- Source directory doesn't exist

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.