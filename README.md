# Kubernetes Odoo with postgres master replica

## Deployment

### First Step Git Clone

not have cluster issuer
```shell
vim cluster-issuer.yaml
```
```yaml
# Kubernetes Odoo Chart — Developer Guide

This repository contains a Helm chart for deploying Odoo with PostgreSQL (master/replica), Redis, and optional autoscaling (HPA/VPA). The guide below helps developers install, configure, and override chart values during development or CI flows.

**Contents**
- **Repository**: Helm chart root with `Chart.yaml`, `values.yaml`, `values.example.yaml`.
- **Templates**: Kubernetes manifests under `templates/` (deployments, services, PVCs, HPA, VPA templates, Postgres statefulsets, Redis, ingress, etc.).

**Prerequisites**
- **Helm 3**: Install Helm on your machine.
- **Kubernetes cluster**: v1.24+ recommended.
- **Optional components**: `cert-manager` (for TLS), a CSI plugin for object storage if using DigitalOcean Spaces.

**Quick Start**

1. Copy the example values into an editable file:

```
cp values.example.yaml values.yaml
```

2. Edit `values.yaml` to set image tags, credentials, and enable/disable features (HPA/VPA) as desired.

3. Install or upgrade the chart:

```
helm upgrade --install <release-name> . --create-namespace -n <namespace> -f values.yaml
```markdown
**Enabling Horizontal Pod Autoscaler (HPA)**
- **Where**: HPA settings live under the chart values, e.g. `odoo.hpa` or a top-level `hpa` section depending on the chart's structure.
- **Typical values snippet** (put this in `values.yaml` or a small override file):

```yaml
odoo:
  hpa:
    enabled: true
    minReplicas: 1
    maxReplicas: 5
    targetCPUUtilizationPercentage: 60
```

- **Notes**: HPA requires the Metrics Server to be installed in the cluster.

**Enabling Vertical Pod Autoscaler (VPA)**
- **Where**: VPA settings live under something like `odoo.vpa` in `values.yaml`.
- **Typical values snippet**:

```yaml
odoo:
  vpa:
    enabled: false
    updateMode: "Auto" # or "Recreate" / "Off"
    minAllowed:
      cpu: 100m
      memory: 128Mi
    maxAllowed:
      cpu: 2
      memory: 4Gi
```

- **Notes**: The cluster must have the VPA admission controller and CRDs installed (the `vertical-pod-autoscaler` components). VPA may not be supported on every managed Kubernetes environment.

**Value Overwrite Flow (developer-friendly options)**

When developing or running CI, you often need to override chart values without editing `values.yaml` directly. Use these options (ordered by priority):

- **`--set`**: Quick, single-value overrides on the CLI.
  - Example: ``helm upgrade --install my-release . -n my-ns --set odoo.replicas=2``
- **`-f` / `--values`**: Provide one or more YAML files with overrides. These are preferred for repeatability.
  - Example: ``helm upgrade --install my-release . -n my-ns -f ci/values.dev.yaml -f values.yaml``
- **`--set-file`**: Set a value using the contents of a file (useful for certificates or large JSON blobs).
- **`--set-string` / `--set-json`**: Force strings or JSON values for tricky types.

Priority (highest → lowest): `--set` / `--set-file` → `--values` files in the order provided → chart `values.yaml` → chart defaults.

Example: enable HPA via `--set` while reusing `values.yaml`:

```
helm upgrade --install odoo-prod . -n production -f values.yaml --set odoo.hpa.enabled=true --set odoo.hpa.maxReplicas=6
```

Or use a dedicated override file for feature flags (recommended for CI):

```
# ci/values.hpa.yaml
odoo:
  hpa:
    enabled: true
    minReplicas: 2
    maxReplicas: 6

helm upgrade --install odoo-ci . -n ci -f values.yaml -f ci/values.hpa.yaml
```

**Inspecting the resolved values**
- Dump the merged values Helm will use:

```
helm get values <release> -n <namespace> --all
helm template . -f values.yaml --set odoo.hpa.enabled=true
```

**Templates and Helpful Commands**
- Render templates locally to verify the output before applying:

```
helm template my-release . -f values.yaml | kubectl apply -f - --dry-run=client
```

- Install `cert-manager` ClusterIssuer for TLS (example):

```
# Example: edit cluster-issuer.yaml then apply
kubectl apply -f cluster-issuer.yaml
```

**Troubleshooting**
- **Pods not scaling**: Verify Metrics Server (for HPA) or VPA components are installed.
- **Values not applied**: Check override priority; use `helm get values` to confirm active values.
- **Ingress/TLS issues**: Confirm the `ingressClassName` and `ClusterIssuer` names match your cluster configuration.

**Chart Files of Interest**
- **`values.example.yaml`**: Example defaults — copy and customize.
- **`templates/odoo-hpa.yaml`**: HPA template.
- **`templates/odoo-vpa.yaml`**: VPA template.
- **`templates/*`**: Other templates for Postgres, Redis, PVCs, services, and ingress.

**Developer Tips**
- Use `helm upgrade --install --wait` in CI to fail fast on deployment issues.
- Keep environment-specific overrides in `ci/` or `environments/` directories and apply them with `-f`.
- Use `helm diff` plugin to preview changes before applying in production.

If you'd like, I can also:
- Add example `ci/values.*.yaml` override files for HPA/VPA.
- Generate a short `helm` cheatsheet with common commands used by your team.

---
## Contact

- email - phonehtutkhaung.dev@gmail.com
- telegram - https://t.me/@phonehtutkhaung
