# Jenkins End-to-End CI/CD Pipeline

Jenkins builds, tests and scans the app, pushes the image to Docker Hub, and commits the new tag to `k8s/`. Argo CD syncs it to the cluster.

## Pipeline

![Jenkins pipeline](pipeline.image.webp)

![Jenkins pipeline stages](pipeline2.image.webp)

![Jenkins pipeline](desk1.webp)

