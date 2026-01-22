---
date:
  created: 2026-01-22
tags:
  - k8s
  - GitOps
  - Secret management
categories:
  - Homelab
---

# Learnings: Installing the 1Password Operator with Flux

A personal note documenting what I learned while setting up the 1Password Connect operator in my k3s homelab.

<!-- more -->

While I had tested the operator some time last year on a local k3s installation (thanks to Rancher Desktop), this time I would need to re-visit the 1password setup and "translate" the helm repo add and helm install commands into GitOps-ready manifests.

---

## The Journey

What started as a "no matches for kind OnePasswordItem" error turned into a deep dive through Flux HelmReleases, SOPS encryption, and Helm value injection
patterns.

## Key Learnings

### 1. Flux Kustomization Dependencies

When using Custom Resource Definitions (CRDs) from an operator, the operator must be fully deployed before any resources using those CRDs are applied. In
Flux, this means adding `dependsOn` to ensure ordering:

```yaml
# clusters/staging/apps.yaml
spec:
dependsOn:
- name: infrastructure-controllers  # Ensures CRDs exist first
```

Without this, Flux reconciles in parallel and the dry-run validation fails because the CRD doesn't exist yet.

### 2. HelmRelease valuesFrom with targetPath

The `valuesFrom` field in a HelmRelease lets you inject secret values into Helm chart values without hardcoding them:

```yaml
valuesFrom:
- kind: Secret
name: onepassword-credentials
valuesKey: connect.credentials_base64    # Key in the K8s Secret
targetPath: connect.credentials_base64   # Path in Helm values.yaml
```

This is equivalent to `helm install --set connect.credentials_base64=<value>` but keeps secrets out of git.

### 3. JSON in Secrets: Use Base64 Encoding

Raw JSON in Kubernetes secrets + Helm value injection = parsing nightmares. The multi-line JSON gets interpreted as YAML, causing cryptic errors like:

```
key "\"healthVerifier\":null" has no value (cannot end with ,)
```

**The fix:** Many Helm charts offer a `_base64` variant for exactly this reason. Check the chart's `values.yaml` for alternatives when raw values cause
issues.

### 4. SOPS Shorthand

Instead of the verbose:
```bash
sops --age=AGE_KEY --encrypt --encrypted-regex='^(data|stringData)$' --in-place secret.yaml
```

Just use (when `.sops.yaml` config exists in your repo):
```bash
sops -e -i secret.yaml
```

The config file already has the age key and regex patterns defined. SOPS finds it by walking up the directory tree.

### 5. Command Chaining for Base64 Credentials

A neat pipeline to prepare credentials for the 1Password operator:

```bash
cat 1password-credentials.json | jq -c . | base64
```

- `jq -c .` — compacts JSON to single line
- `base64` — encodes for safe transport

### 6. Debugging Flux HelmReleases

When a HelmRelease is stuck:

```bash
# Check HelmRelease status
flux get helmrelease -n <namespace>

# Get detailed error message
kubectl describe helmrelease -n <namespace> <name>

# Check Helm controller logs
kubectl logs -n flux-system deployment/helm-controller --tail=50 | grep -i <release-name>
```

### 7. Viewing Decoded Secrets in k9s

In k9s, navigate to a secret and press `x` to decode and view the raw values. Much faster than `kubectl get secret -o yaml | base64 -d` gymnastics.

---

## Files Modified

- `infrastructure/controllers/base/onepassword/release.yaml`: HelmRelease with valuesFrom
- `infrastructure/controllers/base/onepassword/onepassword-credentials.yaml`: SOPS-encrypted credentials
- `clusters/staging/apps.yaml`: Added dependsOn for CRD ordering

## The Operator Should Live in Its Own Namespace

Keep operators in dedicated namespaces (like `onepassword`), not in `flux-system`. The Flux namespace is for Flux components only. Dedicated namespaces
provide better isolation and cleaner RBAC boundaries.
