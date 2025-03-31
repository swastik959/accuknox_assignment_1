# Cow wisdom web server

## Prerequisites

```
sudo apt install fortune-mod cowsay -y
```

## How to use?

1. Run `./wisecow.sh`
2. Point the browser to server port (default 4499)

## Using Docker 

```
docker build -t swastikgour/wisecow .
docker run -p 4499:4499 swastikgour/wisecow

```

## Using Local Kind Setup and self signed certificate

## 🛠️ Prerequisites
Ensure you have the following installed:
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Helm](https://helm.sh/docs/intro/install/)
- [Docker](https://docs.docker.com/get-docker/)

---

## 🚀 Step 1: Setup Kind Cluster
Create a Kind cluster:
```sh
kind create cluster --name kind
```

Verify the cluster:
```sh
kubectl cluster-info --context kind-kind
```

---

## 🌐 Step 2: Install Ingress Controller
Install **NGINX Ingress Controller** using Helm:
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install quickstart-ingress ingress-nginx/ingress-nginx \
  --set controller.service.type=NodePort
```

Ensure Ingress is running:
```sh
kubectl get pods -n default
```

---

## 🔑 Step 3: Configure Self-Signed TLS Certificate

### Install `mkcert`
```sh
sudo apt install libnss3-tools -y
curl -fsSL https://github.com/FiloSottile/mkcert/releases/download/v1.4.3/mkcert-v1.4.3-linux-amd64 -o mkcert
chmod +x mkcert
sudo mv mkcert /usr/local/bin/
```

### Generate Certificates
```sh
mkcert -install
mkcert -cert-file tls.crt -key-file tls.key wisecow.example.com
```

### Create Kubernetes Secret for TLS
```sh
kubectl create secret tls wisecow-tls-cert --cert=tls.crt --key=tls.key
```

---

## 📌 Step 4: Update `/etc/hosts`
Find Kind node IP:
```sh
kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}'
```

Update `/etc/hosts` (replace `<NODE_IP>` with actual IP):
```sh
echo "<NODE_IP> wisecow.example.com" | sudo tee -a /etc/hosts
```

---

## 📦 Step 5: Prepare Kubernetes Manifests
```
kubectl apply -f k8s/wisecow-deployment.yaml
kubectl apply -f k8s/wisecow-service.yaml
kubectl apply -f k8s/wisecow-ingress-tls.yaml
```

## ✅ Step 6: Verify Deployment
Check that everything is running:
```sh
kubectl get pods -A
kubectl get svc
kubectl get ingress -A
```

You can check your application on `https://wisecow.example.com:<Port Number of ingress controller for https>`
