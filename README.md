# Jenkins End-to-End CI/CD Pipeline

Jenkins builds, tests and scans the app, pushes the image to Docker Hub, and commits the new tag to `k8s/`. Argo CD syncs it to the cluster, and Prometheus, Grafana and Alertmanager handle monitoring and alerts.

## Jenkins pipeline

![Jenkins pipeline](pipeline.image.webp)

![Jenkins pipeline stages](pipeline2.image.webp)

## GitOps deployment (Argo CD)

Argo CD watches the `k8s/` folder and syncs the app automatically. The application is Healthy and Synced, with the newest ReplicaSet running.

![Argo CD application](desk1.webp)

## Monitoring

Prometheus scrapes both app pods through the ServiceMonitor.

![Prometheus targets](prometheus-targets.webp)

Grafana dashboard for replicas, CPU usage, pod restarts and target status.

![Grafana dashboard](grafana-dashboard.webp)

## Alerting

Alertmanager sends firing and resolved alerts to Slack.

![Slack alerts](slack-alerts.webp)
