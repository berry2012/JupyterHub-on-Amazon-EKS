# JupyterHub on Amazon EKS

A comprehensive Terraform-based solution for deploying JupyterHub on Amazon EKS with support for GPU workloads, AWS Neuron (Inferentia/Trainium), and auto-scaling capabilities.

## Overview

This project provisions a production-ready JupyterHub environment on Amazon EKS with the following key features:

- **Multi-compute support**: CPU, GPU (G5 instances), AWS Inferentia, and AWS Trainium
- **Auto-scaling**: Karpenter for dynamic node provisioning and Cluster Autoscaler
- **Authentication**: Support for Cognito, OAuth, or dummy authentication
- **Storage**: EFS-based persistent storage for user notebooks and shared data
- **Monitoring**: Prometheus, Grafana, and cost monitoring with Kubecost
- **Security**: IRSA (IAM Roles for Service Accounts) and proper network isolation

## Architecture

The solution deploys:

- **EKS Cluster** with managed node groups and Karpenter auto-scaling
- **JupyterHub** with multiple notebook profiles for different compute types
- **Storage**: EFS for persistent user data and shared storage
- **Networking**: VPC with public/private subnets and secondary CIDR blocks
- **Monitoring**: Prometheus/Grafana stack with FluentBit logging
- **GPU Support**: NVIDIA device plugin with time-slicing and MIG support
- **AWS Neuron**: Support for Inferentia and Trainium instances

## Prerequisites

- AWS CLI configured with appropriate permissions
- Terraform >= 1.0
- kubectl
- Docker (for building custom images)
- Helm (installed automatically by Terraform)

## Quick Start

1. **Clone and navigate to the project**:
   ```bash
   cd /path/to/jupyterhub
   ```

2. **Configure variables** (optional):
   ```bash
   cp terraform.tfvars.example terraform.tfvars
   # Edit terraform.tfvars with your desired configuration
   ```

3. **Deploy the infrastructure**:
   ```bash
   chmod +x install.sh
   ./install.sh
   ```

4. **Access JupyterHub**:
   ```bash
   kubectl port-forward svc/proxy-public 8080:80 -n jupyterhub
   ```
   Open http://localhost:8080 in your browser.

## Configuration

### Authentication Methods

The solution supports three authentication mechanisms:

- **dummy**: Simple username-only authentication (default, for testing)
- **cognito**: AWS Cognito integration with hosted UI
- **oauth**: Generic OAuth2/OIDC provider integration

Configure via the `jupyter_hub_auth_mechanism` variable.

### Notebook Profiles

Pre-configured profiles include:

- **Elyra (CPU)**: General-purpose notebooks with Elyra extensions
- **Data Engineering**: PySpark notebooks with multiple Spark versions
- **GPU Time-Slicing**: Shared GPU access on G5 instances
- **GPU MIG**: Multi-Instance GPU on P4d instances
- **Inferentia**: AWS Inferentia for ML inference
- **Trainium**: AWS Trainium for ML training

## Custom Docker Images

Build and push custom notebook images:

```bash
cd examples
chmod +x create_image.sh
./create_image.sh
```

Available Dockerfiles:
- `jupyterhub-pytorch-neuron.Dockerfile`: PyTorch with Neuron support
- `jupyterhub-tensorflow-neuron.Dockerfile`: TensorFlow with Neuron support
- `elyra-updated-jupyterhub.Dockerfile`: Elyra with updated dependencies

## Monitoring and Observability

### Grafana Dashboard
```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 8080:80 -n kube-prometheus-stack
```
- Username: `admin`
- Password: Retrieved from AWS Secrets Manager

### Kubecost
```bash
kubectl port-forward svc/kubecost-cost-analyzer 9090:9090 -n kubecost
```

## File Structure

```
├── main.tf                 # Main Terraform configuration
├── variables.tf            # Input variables
├── outputs.tf             # Output values
├── vpc.tf                 # VPC and networking
├── jupyterhub.tf          # JupyterHub-specific resources
├── addons.tf              # EKS add-ons and monitoring
├── cognito.tf             # Cognito authentication setup
├── versions.tf            # Terraform and provider versions
├── install.sh             # Deployment script
├── cleanup.sh             # Cleanup script
├── examples/              # Docker images and examples
│   ├── docker/           # Custom Dockerfile definitions
│   ├── notebook-examples/ # Sample Jupyter notebooks
│   └── test-pods/        # Kubernetes test manifests
└── helm/                 # Helm chart configurations
    ├── jupyterhub/       # JupyterHub values files
    ├── efs/              # EFS storage configuration
    └── monitoring/       # Monitoring stack configurations
```

## Customization

### Adding New Notebook Profiles

Edit `helm/jupyterhub/jupyterhub-values-*.yaml` to add new profiles:

```yaml
- display_name: "Custom Profile"
  description: "Custom notebook environment"
  kubespawner_override:
    image: your-custom-image:tag
    node_selector:
      NodePool: default
    cpu_guarantee: 2
    mem_guarantee: 8G
```

### Scaling Configuration

Modify Karpenter node pools in `addons.tf`:

```hcl
limits:
  cpu: 1000  # Adjust based on requirements
```

## Troubleshooting

### Common Issues

1. **Pod stuck in Pending**: Check node capacity and Karpenter provisioning
2. **Image pull errors**: Verify ECR permissions and image availability
3. **Storage issues**: Check EFS mount targets and security groups
4. **Authentication failures**: Verify Cognito/OAuth configuration

### Useful Commands

```bash
# Check JupyterHub pods
kubectl get pods -n jupyterhub

# View Karpenter logs
kubectl logs -f deployment/karpenter -n karpenter

# Check node provisioning
kubectl get nodes --show-labels

# View user pod logs
kubectl logs <user-pod-name> -n jupyterhub
```

## Cleanup

To destroy all resources:

```bash
chmod +x cleanup.sh
./cleanup.sh
```

## Security Considerations

- EFS encryption at rest is enabled
- Network isolation with private subnets
- IRSA for fine-grained AWS permissions
- Security groups restrict access appropriately
- Secrets stored in AWS Secrets Manager

## Cost Optimization

- Karpenter automatically scales nodes based on demand
- Spot instances supported for cost savings
- EFS Intelligent Tiering for storage optimization
- Kubecost provides detailed cost visibility

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and questions:
- Check the troubleshooting section
- Review AWS EKS and JupyterHub documentation
- Open an issue in the repository