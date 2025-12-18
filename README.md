# Helm-chart-k8s
Helm chart for k8s deployments
# Backend Application Helm Chart

This Helm chart deploys a containerized backend application on Kubernetes with flexible configuration for networking, scaling, scheduling, and environment-specific deployment strategies. It is designed to work well on **Amazon EKS**, supporting **AWS ALB Ingress**, **NGINX Ingress**, autoscaling, and common production requirements.

---

## Overview

This chart provides:

* Deployment of a backend container from **AWS ECR**
* Configurable **Service** (ClusterIP / Headless)
* **AWS ALB Ingress** or **NGINX Ingress** support
* Horizontal Pod Autoscaler (HPA)
* Vertical Pod Autoscaler (VPA)
* ConfigMaps for files and environment variables
* Node affinity, tolerations, and node selectors
* Liveness & readiness probes
* RollingUpdate or Recreate deployment strategies
* Git metadata injection for traceability

---

## Prerequisites

* Kubernetes v1.32+
* Helm v3.x
* (For ALB) AWS Load Balancer Controller installed
* (For NGINX) NGINX Ingress Controller installed
* Access to AWS ECR (if using private images)

---

## Chart Structure

```
helm-base-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   ├── vpa.yaml
│   └── configmap.yaml
└── README.md
```

---

## Installation

### Install the chart

```bash
helm install backend-app ./helm-base-chart \
  --namespace dev \
  --create-namespace
```

### Upgrade

```bash
helm upgrade backend-app ./helm-base-chart -n dev
```

### Uninstall

```bash
helm uninstall backend-app -n dev
```
---

## Configuration
All configuration is controlled via `values.yaml`.
---

## Image Configuration

```yaml
image:
  repository: <aws-ecr-repo>
  tag: be_v-5
  pullPolicy: IfNotPresent
```

| Field      | Description                  |
| ---------- | ---------------------------- |
| repository | Docker image repository      |
| tag        | Image tag                    |
| pullPolicy | Kubernetes image pull policy |

---
## Service Configuration

```yaml
service:
  enabled: true
  type: ClusterIP
  headless: false
  port: 80
  targetPort: 6000
```

| Field      | Description                        |
| ---------- | ---------------------------------- |
| enabled    | Enable or disable Service creation |
| type       | Kubernetes Service type            |
| headless   | Enable headless service            |
| port       | Service port                       |
| targetPort | Container port                     |

---

## Resource Management

```yaml
resources:
  limits:
    memory: 128Mi
    cpu: 500m
  requests:
    memory: 64Mi
    cpu: 250m
```

Defines CPU and memory requests and limits for the container.

---

## Image Pull Secrets

```yaml
imagePullSecrets:
  enabled: false
  values:
    - name: credreg-1
```

Enable this when pulling images from private registries.

---
## Special Features

### Git Metadata

```yaml
gitBranch: master
gitCommit: "HEAD"
gitBuildNumber: "10"
gitBuildTimestamp: "20241010"
```

These values can be injected as labels or environment variables for:

* Debugging
* Deployment traceability
* Observability

### Affinity Rules

```yaml
affinity:
  enabled: false
```

Supports:

* Required and preferred node affinity
* Spot / on-demand node scheduling

### Custom Container Command

```yaml
command:
  enabled: false
```

Override the container entrypoint when required.


### Multi-Ingress Configuration (AWS ALB & Nginx)

### AWS ALB Ingress

```yaml
ingress:
  enabled: true
  className: alb
```

Features:

* HTTPS via ACM
* ALB group support
* Internet-facing or internal
* Path-based routing

| Field       | Description              |
| ----------- | ------------------------ |
| annotations | ALB-specific annotations |
| rules       | Host and path rules      |
| tls         | TLS configuration        |

---

### NGINX Ingress (Alternative)

```yaml
nginx:
  enabled: false
```

Use this when deploying with the NGINX ingress controller instead of ALB.

### Autoscaling (HPA)

```yaml
autoscaling:
  enabled: false
```

Supports:

* CPU-based scaling
* Memory-based scaling

| Field       | Description      |
| ----------- | ---------------- |
| minReplicas | Minimum replicas |
| maxReplicas | Maximum replicas |


### Vertical Pod Autoscaler (VPA)

```yaml
vpa:
  enabled: false
  updateMode: "Off"
```

Controls automatic resource tuning.

---

### ConfigMap Support i.e, configmaps can be mounted as volume

```yaml
configmap:
  enabled: false
```

Supports:

* File-based configuration
* Environment variables

Useful for application configuration without rebuilding images.

### Node Selector

```yaml
nodeSelector:
  enabled: false
```

Restrict pods to specific nodes.


### Tolerations

```yaml
tolerations:
  enabled: false
```

Allow scheduling onto tainted nodes.


### Health Probes

### Liveness Probe

```yaml
livenessProbe:
  enabled: false
```

### Readiness Probe

```yaml
readinessProbe:
  enabled: false
```

Improves application reliability and rollout safety.

### Deployment Strategy

### Lower Environment Strategy

```yaml
lowerenvStrategy:
  enabled: true
```

Uses **Recreate** strategy for dev/test environments.

### Default Strategy

```yaml
strategy:
  type: RollingUpdate
  maxUnavailable: 50%
  maxSurge: 1
```

Recommended for staging and production.


## Example Override

```bash
helm install backend-app ./helm-base-chart \
  --set image.tag=be_v-6 \
  --set autoscaling.enabled=true
```

## Best Practices

* Use `RollingUpdate` for production
* Enable probes for zero-downtime deployments
* Use Spot enablement for cost cutting
* Use HPA for variable traffic
* Use ConfigMaps for non-secret configs
* Track builds using Git metadata
