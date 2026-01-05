# MeshLogic Helm Charts

Official Helm charts for deploying MeshLogic agents on Kubernetes.

## Usage

### Add the Helm Repository

```bash
helm repo add meshlogic https://meshlogic-pty-ltd.github.io/meshlogic-helm-charts
helm repo update
```

### Install the Agent

```bash
# Create namespace and credentials secret
kubectl create namespace meshlogic
kubectl create secret generic meshlogic-credentials -n meshlogic \
    --from-literal=api-key=YOUR_API_KEY

# Install the chart
helm install meshlogic-agent meshlogic/meshlogic-agent \
    --namespace meshlogic \
    --set agent.customerId=YOUR_CUSTOMER_ID
```

## Available Charts

| Chart | Description |
|-------|-------------|
| [meshlogic-agent](./charts/meshlogic-agent) | MeshLogic eBPF behavioral monitoring agent |

## Configuration

See the individual chart READMEs for configuration options:

- [meshlogic-agent values](./charts/meshlogic-agent/README.md)

## Requirements

- Kubernetes 1.23+
- Helm 3.8+
- Linux nodes with kernel 5.4+ (for eBPF support)
- Privileged pods enabled (for BPF capabilities)

## Image Access

The MeshLogic agent images are hosted in a private registry. You will need:

1. A valid MeshLogic license
2. Registry credentials (provided during onboarding)

```bash
# Create image pull secret
kubectl create secret docker-registry meshlogic-registry \
    --namespace meshlogic \
    --docker-server=registry.meshlogic.ai \
    --docker-username=YOUR_USERNAME \
    --docker-password=YOUR_PASSWORD
```

## Support

- Documentation: https://meshlogic.ai/docs
- Issues: https://github.com/MeshLogic-Pty-Ltd/meshlogic-helm-charts/issues
- Email: support@meshlogic.ai

## License

Apache License 2.0 - see [LICENSE](LICENSE) for details.
