# Kustomize-based Kubernetes Deployment

This directory contains Kubernetes manifests structured with Kustomize for 
managing multiple environments of the same application.

## Directory Structure

```
local_argocd/k8s/
├── base/                    # Base configuration (shared by all envs)
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── namespace.yaml
│   ├── pv.yaml
│   └── pvc.yaml
└── overlays/               # Environment-specific configurations
    ├── dev/
    │   ├── dev1/          # Dev instance 1
    │   │   └── kustomization.yaml
    │   └── dev2/          # Dev instance 2
    │       └── kustomization.yaml
    └── preprod/           # Pre-production environment
        └── kustomization.yaml
```

## Environment Specifications

### Dev1 (php-app-dev1)
- **Namespace:** php-app-dev1
- **Replicas:** 1
- **Storage:** 50Mi
- **Host:** php-app-dev1.test
- **Resources:** Default (memory: 128Mi, cpu: 200m)

### Dev2 (php-app-dev2)
- **Namespace:** php-app-dev2
- **Replicas:** 1
- **Storage:** 50Mi
- **Host:** php-app-dev2.test
- **Resources:** Default (memory: 128Mi, cpu: 200m)

### Preprod (php-app-preprod)
- **Namespace:** php-app-preprod
- **Replicas:** 2
- **Storage:** 200Mi
- **Host:** php-app-preprod.test
- **Resources:** Increased (memory: 256Mi, cpu: 500m)

## Usage

### Testing manifests locally

Build and view the generated manifests for each environment:

```bash
# Dev1
kubectl kustomize overlays/dev/dev1

# Dev2
kubectl kustomize overlays/dev/dev2

# Preprod
kubectl kustomize overlays/preprod
```

### Applying directly with kubectl

```bash
# Deploy dev1
kubectl apply -k overlays/dev/dev1

# Deploy dev2
kubectl apply -k overlays/dev/dev2

# Deploy preprod
kubectl apply -k overlays/preprod
```

### Using with Argo CD

Argo CD applications are defined in `../`:
- `application-dev1.yaml` - Points to k8s/overlays/dev/dev1
- `application-dev2.yaml` - Points to k8s/overlays/dev/dev2
- `application-preprod.yaml` - Points to k8s/overlays/preprod

Deploy all applications:
```bash
kubectl apply -f ../application-dev1.yaml
kubectl apply -f ../application-dev2.yaml
kubectl apply -f ../application-preprod.yaml
```

## Accessing the Applications

After deployment, add these entries to your `/etc/hosts`:

```
127.0.0.1 php-app-dev1.test
127.0.0.1 php-app-dev2.test
127.0.0.1 php-app-preprod.test
```

Then access:
- Dev1: http://php-app-dev1.test
- Dev2: http://php-app-dev2.test
- Preprod: http://php-app-preprod.test

## Customizing Overlays

To add a new environment or modify existing ones, edit the 
`kustomization.yaml` in the respective overlay directory. Common patches:

- **Replicas:** Adjust pod count
- **Resources:** Modify CPU/memory limits
- **Storage:** Change PV/PVC size
- **Hostname:** Update ingress host
- **Labels:** Add environment-specific labels

