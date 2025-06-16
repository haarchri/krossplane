# Install

```bash
task crossplane-with-ack
task crossplane-with-ack-cleanup
```

```bash
task crossplane-with-upjet
task crossplane-with-upjet-cleanup
```

```bash
task kro-with-ack
task kro-with-ack-cleanup
```

```bash
task kro-with-upjet
task kro-with-upjet-cleanup
```

# ACK Controllers and Crossplane Providers

## Overview Comparison

| Aspect | ACK Controllers | Crossplane Providers |
|--------|----------------|---------------------|
| **Installation** | Helm Chart | Crossplane Package Manager |
| **Authentication** | IRSA, PodIdentity | IRSA, PodIdentity |
| **Region Strategy** | Hierarchical (resource → namespace → cluster) | Per-resource via ProviderConfig |
| **Cross-Account** | ConfigMap + namespace annotations | Per-resource via ProviderConfig |

## Region Configuration

| Feature | ACK Controllers | Crossplane Providers |
|---------|----------------|---------------------|
| **Default Region** | Cluster-wide via `--aws-region` flag |  No default, must specify per resource |
| **Namespace-level** | `services.k8s.aws/region` annotation | Not supported |
| **Resource-level** | `services.k8s.aws/region` annotation | `spec.forProvider.region` field |
| **Precedence Order** | Resource → Namespace → Cluster | Direct specification in `spec.forProvider.region` |

## Cross-Account Management

| Feature | ACK Controllers | Crossplane Providers |
|---------|----------------|---------------------|
| **Account Mapping** | ConfigMap with account-to-role mapping | Individual ProviderConfig per account/auth method |
| **Namespace Binding** | `services.k8s.aws/owner-account-id` annotation | Per-resource via ProviderConfig |
| **Centralized Config** | One ConfigMap for all accounts |  Multiple ProviderConfigs |
| **Resource Override** | Can override namespace default | Specify different ProviderConfig |

## Configuration Examples

### ACK Controllers
```yaml
# Cluster-wide default (Helm values)
aws:
  region: us-east-1

# Namespace-level default
apiVersion: v1
kind: Namespace
metadata:
  name: production
  annotations:
    services.k8s.aws/region: eu-west-1
    services.k8s.aws/owner-account-id: "111111111111"

# Resource-level override
apiVersion: s3.services.k8s.aws/v1alpha1
kind: Bucket
metadata:
  annotations:
    services.k8s.aws/region: ap-southeast-1
```

### Crossplane Providers
```yaml
# ProviderConfig (for auth only)
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: aws-prod
spec:
  credentials:
    source: IRSA

# Managed Resource (region specified here)
apiVersion: s3.aws.crossplane.io/v1beta1
kind: Bucket
spec:
  forProvider:
    region: us-east-1
  providerConfigRef:
    name: aws-prod
```

## Trade-offs Summary

| Consideration | ACK Controllers | Crossplane Providers |
|--------------|----------------|---------------------|
| **Ease of Setup** | Simpler defaults and inheritance | More explicit configuration required |
| **Flexibility** | Multiple configuration levels | Fine-grained per-resource control |
| **Maintenance** | Less Configuration to manage | More ProviderConfigs needed |
| **Explicitness** | Config inheritance can be unclear | Very explicit per resource |
