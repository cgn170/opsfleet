# AWS EKS Cluster Setup with Terraform

## Prerequisites

- AWS CLI
- Terraform
- kubectl

## Setup Instructions

```bash
# Clone the repository
git clone https://github.com/cgn170/opsfleet.git
cd opsfleet/eks-cluster

# Initialize Terraform
terraform init

# Run Terraform plan and deploy the EKS cluster
terraform plan
terraform apply --auto-approve
```

## Post-Deployment Steps

```bash
# Update kubeconfig to have access to the new EKS cluster
aws eks --region us-east-1 update-kubeconfig --name opsfleet-eks-cluster

# Check the status of the karpenter controller
kubectl logs -f -n kube-system -l app.kubernetes.io/name=karpenter -c controller

# Test karpenter configuration
# Deploy example files
kubectl apply -f inflate-amd.yaml
kubectl apply -f inflate-arm.yaml

# Check the status of the nodes
kubectl get nodes -L karpenter.sh/registered

# Remove example files
kubectl delete -f inflate-arm.yaml
kubectl delete -f inflate-amd.yaml
```

## Cleanup

```bash
# Destroy the EKS cluster
terraform destroy --auto-approve
```

# How to run a deployment in arm64 or amd64 nodes

Note: Nodes has been labeled with `arch: arm64` and `arch: amd64` respectively and tolerations have been added to the deployments to ensure they only run on the correct architecture.

### Example for amd64 nodes:

```bash
# Deployment example for arm64 nodes
cat << EOF > example-amd.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-amd
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-amd
  template:
    metadata:
      labels:
        app: nginx-amd
    spec:
      nodeSelector: # Add node selector for amd64
        arch: amd64 # Add this label to target amd64 nodes
      terminationGracePeriodSeconds: 0
      containers:
        - name: nginx-amd
          image: nginx
          resources:
            requests:
              cpu: 1
      tolerations:  # Add tolerations for amd64 nodes
        - key: "arch"
          value: "amd64"
          effect: "NoSchedule"
EOF


```

Example for arm64 nodes:

```bash
### Deployment example for arm64 nodes
cat << EOF > example-arm.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-arm
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-arm
  template:
    metadata:
      labels:
        app: nginx-arm
    spec:
      nodeSelector: # Add node selector for arm64
        arch: arm64 # Add this label to target arm64 nodes
      terminationGracePeriodSeconds: 0
      containers:
        - name: nginx-arm
          image: nginx
          resources:
            requests:
              cpu: 1
      tolerations:  # Add tolerations for arm64 nodes
        - key: "arch"
          value: "arm64"
          effect: "NoSchedule"
EOF

```
