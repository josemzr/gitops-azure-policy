# GitOps Azure Policy

This repository contains policies and configurations to ensure that namespaces in Kubernetes clusters have NetworkPolicies that restrict pod communication to the same namespace.

## Repository Components

- `azure-policies/`: Azure Policy definitions for auditing AKS clusters
- `network-policies/`: Kubernetes resources for network policy enforcement
  - `gatekeeper/`: Gatekeeper templates and constraints
  - `templates/`: NetworkPolicy templates
  - `cluster-policy/`: Kustomize configuration to apply NetworkPolicies

## Implementing with GitOps

### 1. Installation and configuration of components

1. **Install Gatekeeper in your cluster**:
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.9/deploy/gatekeeper.yaml
   ```

2. **Install the GitOps infrastructure** (e.g., Flux or ArgoCD)

   - For Flux:
   ```bash
   flux bootstrap github \
     --owner=<GITHUB_USER> \
     --repository=gitops-azure-policy \
     --branch=main \
     --path=./clusters/my-cluster \
     --personal
   ```

   - For ArgoCD:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

### 2. Deploy Kubernetes resources

Add the following definitions for your GitOps tool to sync Gatekeeper components and NetworkPolicies:

```yaml
# For Flux
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: network-policies
  namespace: flux-system
spec:
  interval: 10m0s
  path: ./network-policies
  prune: true
  sourceRef:
    kind: GitRepository
    name: gitops-flux-source
```

### 3. Apply the Azure Policy

1. Create a custom policy definition in Azure Policy using the `azure-policies/require-same-namespace-networkpolicy.json` file
2. Assign the policy to your subscriptions or management groups containing AKS clusters

### 4. Complete workflow

1. Developers create new namespaces
2. Azure Policy audits that namespaces have an appropriate NetworkPolicy
3. The GitOps tool automatically applies default NetworkPolicies to all namespaces
4. Gatekeeper validates new namespaces in real-time to ensure compliance

## Adapting to your needs

- Modify the NetworkPolicy template in `network-policies/templates/default-networkpolicy.yaml` according to your specific requirements
- Update namespace exclusions in the policy definition and Gatekeeper constraints