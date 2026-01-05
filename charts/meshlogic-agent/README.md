# MeshLogic Agent Helm Chart

Deploys the MeshLogic eBPF behavioral monitoring agent as a DaemonSet on Kubernetes.

## Prerequisites

- Kubernetes 1.23+
- Helm 3.8+
- Linux nodes with kernel 5.4+ (BTF support required)
- Privileged pod execution enabled

## Installation

```bash
# Add the MeshLogic Helm repository
helm repo add meshlogic https://meshlogic-pty-ltd.github.io/meshlogic-helm-charts
helm repo update

# Create namespace
kubectl create namespace meshlogic

# Create image pull secret
kubectl create secret docker-registry meshlogic-registry \
    --namespace meshlogic \
    --docker-server=registry.meshlogic.ai \
    --docker-username=YOUR_USERNAME \
    --docker-password=YOUR_PASSWORD

# Install the chart
helm install meshlogic-agent meshlogic/meshlogic-agent \
    --namespace meshlogic \
    --set agent.customerId=YOUR_CUSTOMER_ID
```

## Configuration

### Required Values

| Parameter | Description |
|-----------|-------------|
| `agent.customerId` | Your MeshLogic customer ID |

### Common Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `agent.safetyTier` | Safety tier (1-3) | `2` |
| `agent.region` | AWS region for telemetry | `ap-southeast-2` |
| `image.tag` | Image tag | Chart appVersion |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `256Mi` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `128Mi` |

### AWS Authentication

#### Using IRSA (Recommended)

```yaml
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/meshlogic-agent

aws:
  useIRSA: true
```

#### Using Credentials Secret

```bash
kubectl create secret generic aws-credentials -n meshlogic \
    --from-literal=aws-access-key-id=YOUR_KEY \
    --from-literal=aws-secret-access-key=YOUR_SECRET
```

```yaml
aws:
  useIRSA: false
  credentialsSecret: aws-credentials
```

### eBPF Configuration

```yaml
ebpf:
  enableProcess: true
  enableFile: true
  enableNetwork: true
  ringBufferSize: 1048576
  mutePaths:
    - /proc
    - /sys
    - /dev
```

### Privacy Configuration

```yaml
privacy:
  epsilon: 1.0  # Differential privacy parameter
  enableNoise: true
```

### Prometheus Monitoring

```yaml
metrics:
  enabled: true
  port: 9090
  serviceMonitor:
    enabled: true
    interval: 30s
```

## Uninstallation

```bash
helm uninstall meshlogic-agent -n meshlogic
kubectl delete namespace meshlogic
```

## Troubleshooting

### Agent not starting

Check pod status and logs:

```bash
kubectl get pods -n meshlogic
kubectl logs -n meshlogic -l app.kubernetes.io/name=meshlogic-agent
```

### eBPF programs failing to load

Verify kernel version and BTF support:

```bash
kubectl exec -n meshlogic <pod-name> -- uname -r
kubectl exec -n meshlogic <pod-name> -- ls /sys/kernel/btf/vmlinux
```

### AWS connectivity issues

Test from within the pod:

```bash
kubectl exec -n meshlogic <pod-name> -- \
    curl -I https://dynamodb.ap-southeast-2.amazonaws.com
```

## Support

- Documentation: https://meshlogic.ai/docs
- Issues: https://github.com/MeshLogic-Pty-Ltd/meshlogic-helm-charts/issues
- Email: support@meshlogic.ai
