# k3d Cluster Setup

Streamlined k3d Kubernetes cluster deployment with Nginx Ingress Controller and TLS certificate support.

> ⭐ If you find this project helpful, please consider giving it a star on GitLab! It helps others discover the project.

## Overview

This project provides an automated way to create and configure a local Kubernetes cluster using k3d (k3s in Docker). The setup includes:

- **1 Server node** and **2 Agent nodes**
- **Nginx Ingress Controller** (Traefik disabled)
- **TLS certificate configuration** for HTTPS support
- **Shared volume mount** at `/mnt`
- **Port forwarding** for HTTP (80) and HTTPS (443)

## Prerequisites

Before running this project, ensure you have the following installed:

- **Docker** - Required for k3d to run
- **k3d** - Kubernetes in Docker
- **kubectl** - Kubernetes command-line tool
- **Helm** - Package manager for Kubernetes (for Nginx Ingress installation)
- **kubectx** (optional) - For easier context switching

### System Requirements

- Docker daemon running
- At least 4GB of available RAM
- Ports 80, 443, and 6445 available on the host

## Cluster Configuration

The cluster is configured via `config.yml` with the following specifications:

- **Cluster Name**: k3d
- **K3s Version**: v1.33.3-k3s1
- **API Server**: Exposed on `0.0.0.0:6445`
- **Servers**: 1
- **Agents**: 2
- **Load Balancer**: Enabled (ports 80 and 443)
- **Traefik**: Disabled (using Nginx Ingress instead)
- **Volume Mount**: `./mnt:/mnt` (shared across all nodes)

## Installation

### 1. Clone the Repository

```bash
git clone https://gitlab.com/inci-projects/k3d.git
cd k3d
```

### 2. Update Configuration

Modify the `config.yml` file to match your local setup:

**Cluster Name** (line 4):

```yaml
metadata:
  name: your-cluster-name
```

**Volume Mount Path** (line 12):

```yaml
volumes:
  - volume: /your/custom/path:/mnt
    nodeFilters:
      - server:0
      - agent:*
```

**Other optional configurations**:

- Number of servers/agents
- Port mappings
- K3s version

### 3. Create the Cluster

```bash
k3d cluster create --config config.yml
```

### 4. Install Nginx Ingress Controller (Optional)

After the cluster is created, you can install Nginx Ingress Controller:

```bash
# Add Nginx Ingress repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install Nginx Ingress Controller
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace kube-system \
  --set controller.ingressClassResource.name=nginx \
  --wait
```

### 5. Configure TLS Certificates (Optional)

If you want to use HTTPS, create a TLS secret:

```bash
kubectl create secret tls tls-secret \
  --key /path/to/your/privkey.pem \
  --cert /path/to/your/fullchain.pem \
  --namespace kube-system
```

Then reference it in your Helm installation:

```bash
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace kube-system \
  --set controller.ingressClassResource.name=nginx \
  --set controller.extraArgs.default-ssl-certificate=kube-system/tls-secret \
  --wait
```

## Usage

### Access the Cluster

After installation, you can interact with your cluster using kubectl:

```bash
# View cluster nodes
kubectl get nodes

# View all pods
kubectl get pods --all-namespaces

# Switch context (if kubectx is installed)
kubectx <cluster-name>
```

### Deploy Applications

Deploy your applications to the cluster and expose them via Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  tls:
    - hosts:
        - example.local
      secretName: tls-secret
  rules:
    - host: example.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: your-service
                port:
                  number: 80
```

### Access Shared Volume

The `/mnt` directory is mounted from your host to all cluster nodes, allowing you to share files:

```bash
# Place files in the local mnt directory
cp myfile.txt ./mnt/

# Access from within a pod
kubectl exec -it <pod-name> -- ls /mnt
```

## Managing the Cluster

### Stop the Cluster

```bash
k3d cluster stop <cluster-name>
```

### Start the Cluster

```bash
k3d cluster start <cluster-name>
```

### Delete the Cluster

```bash
k3d cluster delete <cluster-name>
```

### View Cluster Info

```bash
k3d cluster list
k3d node list
```

## Troubleshooting

### Cluster Creation Fails

- Ensure Docker is running: `docker ps`
- Check if ports 80, 443, and 6445 are available
- Review cluster creation logs: `k3d cluster list`
- Verify the config.yml file is valid

### TLS Certificate Issues

- Verify the certificate paths are correct and accessible
- Ensure certificates are valid and not expired
- Check secret creation: `kubectl get secrets -n kube-system`

### Ingress Not Working

- Verify Nginx Ingress Controller is running:
  ```bash
  kubectl get pods -n kube-system | grep ingress
  ```
- Check ingress resources:
  ```bash
  kubectl get ingress --all-namespaces
  ```

### Context Issues

If `kubectx` is not installed, manually switch contexts:

```bash
kubectl config use-context k3d-<cluster-name>
```

## Project Structure

```
k3d/
├── config.yml      # k3d cluster configuration
├── mnt/            # Shared volume mount directory
└── README.md       # This file
```

## Support the Project

If you find this project useful, please consider:

- ⭐ **Starring** the repository on GitLab
- 🐛 **Reporting issues** if you find any bugs
- 💡 **Suggesting features** or improvements
- 🔀 **Contributing** via merge requests
- 📢 **Sharing** the project with others

Your support helps make this project better for everyone!

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

This project is provided as-is for educational and development purposes.

## Notes

- This setup is intended for **local development** environments
- For production use, consider additional security hardening and high-availability configurations
- The cluster uses the lightweight k3s distribution, which is ideal for development and edge computing scenarios
