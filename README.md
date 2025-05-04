# KIND Cluster Setup Guide

## 1. Installing KIND and kubectl
Install KIND and kubectl using the provided script:
```bash

#!/bin/bash

[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

VERSION="v1.30.0"
URL="https://dl.k8s.io/release/${VERSION}/bin/linux/amd64/kubectl"
INSTALL_DIR="/usr/local/bin"

curl -LO "$URL"
chmod +x kubectl
sudo mv kubectl $INSTALL_DIR/
kubectl version --client

rm -f kubectl
rm -rf kind

echo "kind & kubectl installation complete."
```

## 2. Setting Up the KIND Cluster
Create a kind-cluster-config.yaml file:

```yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
- role: worker
  image: kindest/node:v1.31.2
```
Create the cluster using the configuration file:

```bash

kind create cluster --config kind-cluster-config.yaml --name my-kind-cluster
```
![image](https://github.com/user-attachments/assets/2a795e81-8a5b-417f-a80a-055abba34e27)

Verify the cluster:

```bash

kubectl get nodes
kubectl cluster-info
```
![image](https://github.com/user-attachments/assets/c0c14db1-666a-49ae-92a6-c57c30c412e1)

## 3. Accessing the Cluster
Use kubectl to interact with the cluster:
```bash

kubectl cluster-info
```


## 4. Setting Up the Kubernetes Dashboard
Deploy the Dashboard
Apply the Kubernetes Dashboard manifest:
```bash

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```
![image](https://github.com/user-attachments/assets/9e10b18d-ae01-46f8-b0b9-748c4cc3afd8)

Create an Admin User
Create a dashboard-admin-user.yml file with the following content:

```yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
Apply the configuration:

```bash

kubectl apply -f dashboard-admin-user.yml
```
![image](https://github.com/user-attachments/assets/736a9b0d-0c30-4104-81a4-44ec7702eebd)
![image](https://github.com/user-attachments/assets/542b1e27-1a15-426d-8538-9c521891d1cb)

Get the Access Token
Retrieve the token for the admin-user:

```bash

kubectl -n kubernetes-dashboard create token admin-user
```
Copy the token for use in the Dashboard login.

Access the Dashboard
Start the Dashboard using kubectl proxy:

```bash

kubectl proxy
```

![image](https://github.com/user-attachments/assets/9670577f-5243-4d4d-90b6-631ae73a49a8)


Open the Dashboard in your browser:

```bash

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```
Use the token from the previous step to log in.
![image](https://github.com/user-attachments/assets/cf74bf9a-00d7-40f6-8419-392c1e82bacb)

![image](https://github.com/user-attachments/assets/5f083d98-2030-4d09-a41b-88d54aa4405e)

Sometimes Docker networking or firewall rules block the port. You can try running:

```bash
sudo ufw allow 8001
```
![image](https://github.com/user-attachments/assets/a0f27891-4e97-4595-b335-d45742a018b1)

## 5. Deleting the Cluster
Delete the KIND cluster:
```bash

kind delete cluster --name my-kind-cluster
```

## 6. Notes

Multiple Clusters: KIND supports multiple clusters. Use unique --name for each cluster.
Custom Node Images: Specify Kubernetes versions by updating the image in the configuration file.
Ephemeral Clusters: KIND clusters are temporary and will be lost if Docker is restarted.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details. 
