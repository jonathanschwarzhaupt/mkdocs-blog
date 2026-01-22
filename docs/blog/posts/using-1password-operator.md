---
date:
  created: 2026-01-23
tags:
  - k8s
  - GitOps
  - Secret management
categories:
  - Homelab
---

# Learnings: Using the 1Password Operator with Flux

A personal note documenting what I learned while migrating Linkding from SOPS secrets to 1Password.

<!-- more -->

## The Challenge

1Password items have standard field names like username and password, but applications often expect specific environment variable names (e.g.,
LD_SUPERUSER_NAME, LD_SUPERUSER_PASSWORD). The OnePasswordItem CRD creates K8s secrets with keys matching the 1Password field names directly—no built-in
field mapping.

Solution: valueFrom.secretKeyRef

Replace envFrom.secretRef (which imports all keys as-is) with explicit env entries using valueFrom.secretKeyRef:

## Before: All secret keys become env vars with same names

```yaml
envFrom:
- secretRef:
    name: linkding-container-env
```

## After: Explicit mapping from secret keys to env var names

```yaml
env:
- name: LD_SUPERUSER_NAME           # Target env var name
  valueFrom:
    secretKeyRef:
      name: linkding-container-env
      key: username                  # Source key from 1Password
- name: LD_SUPERUSER_PASSWORD
  valueFrom:
    secretKeyRef:
      name: linkding-container-env
      key: password
```

## Steps Taken

1: Created the OnePasswordItem Resource

```yaml
apiVersion: onepassword.com/v1
kind: OnePasswordItem
metadata:
  name: linkding-container-env
  annotations:
    operator.1password.io/auto-restart: "true"
spec:
  itemPath: "vaults/apps/items/linkding"
```

The auto-restart annotation tells the operator to restart pods when the secret changes.

2: Updated the Deployment

Modified apps/base/linkding/deployment.yaml to use explicit field mapping instead of envFrom.

3: Updated Kustomization

Removed the old SOPS-encrypted secret from apps/staging/linkding/kustomization.yaml and kept the OnePasswordItem resource.

4: Committed and Pushed

Two commits:
- feat: migrate linkding secret to 1password
- test: change secret name

Debugging: Flux Not Picking Up New Commits

Problem: After pushing and running flux reconcile kustomization apps, Flux applied a revision from 2 commits ago.

Root cause: The GitRepository source hadn't fetched the latest commits yet. Reconciling a Kustomization only re-applies from the cached source.

Solution: Reconcile the source first, then the kustomization:

### Step 1: Fetch latest from git
flux reconcile source git flux-system

### Step 2: Apply the kustomization
flux reconcile kustomization apps

Key insight: Flux has a two-stage pipeline:
1. Source controller fetches from git → caches artifacts
2. Kustomize controller applies from cached artifacts

If you only reconcile the kustomization, it uses whatever revision the source already has cached.

## Verification Commands

### Check what revision Flux has cached
kubectl get gitrepository -n flux-system flux-system -o jsonpath='{.status.artifact.revision}'

### Check what revision each kustomization has applied
kubectl get kustomization -n flux-system -o wide

### Verify the 1Password-generated secret exists
kubectl get secret linkding-container-env -n linkding -o yaml

### Check the linkding pod restarted
kubectl get pods -n linkding

## Files Modified

- apps/base/linkding/deployment.yaml — Changed envFrom to explicit env mapping
- apps/staging/linkding/linkding-container-env-secret.yaml — OnePasswordItem with itemPath
- apps/staging/linkding/kustomization.yaml — Removed SOPS secret reference

## Key Takeaways

1. envFrom vs env: Use envFrom when secret keys match env var names; use env with valueFrom.secretKeyRef when you need to rename them
2. Flux reconciliation order: Source first (flux reconcile source git), then kustomization
3. auto-restart annotation: Add operator.1password.io/auto-restart: "true" to automatically restart pods when secrets change
