Thanks! Based on your update, I’ve incorporated the **EC2 setup as the first step** and added your **Kubernetes Deployment and Service YAML** to the final guide. Here’s the **complete and final version** of your documentation:

---

# 🚀 Prometheus & Grafana Monitoring on EKS using Helm

> 🔧 End-to-end observability made simple — from EC2 setup to visual dashboards in your Kubernetes cluster!

[![Prometheus & Grafana Architecture](https://github.com/laditrinath321/Prometheus-Grafana-setup/blob/main/Architecture.JPG?raw=true)](https://github.com/laditrinath321/Prometheus-Grafana-setup/blob/main/Architecture.JPG)


---

## 📖 Overview

This repository provides a step-by-step guide to **deploy Prometheus and Grafana** on an **Amazon EKS (Elastic Kubernetes Service)** cluster using **Helm 3**. By the end, you'll have:

* 🎯 Real-time monitoring of CPU, memory, errors, traffic, latency & saturation
* 📊 Visual dashboards for metrics, logs & traces
* 🧠 Foundation for full observability and root cause analysis

---

## 🧱 Step 0: Launch EC2 (Amazon Linux)

1. Open AWS Console → Launch EC2 Instance
2. Choose Amazon Linux 2 AMI
3. Instance Type: `t2.micro`
4. Key Pair: Create/Choose existing
5. Allow ports: `22`, `80`, `443`
6. Launch and SSH into it:

```bash
ssh -i "<your-key>.pem" ec2-user@<your-ec2-public-ip>
```

---

## 🔧 Prerequisites (Install on EC2)

Install the following:

| Tool      | Version (Recommended) | Installation                                                                                                                                                                       |
| --------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `awscli`  | v2.x                  | [Install Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)                                                                                               |
| `eksctl`  | Latest                | `curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \| tar xz -C /tmp && sudo mv /tmp/eksctl /usr/local/bin` |
| `kubectl` | Latest                | See below ⬇️                                                                                                                                                                       |
| `helm`    | v3.x                  | See below ⬇️                                                                                                                                                                       |

---

## 🔥 Quick Start Guide

### 📌 Step 1: Install `kubectl`

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

---

### 📌 Step 2: Create an EKS Cluster

```bash
eksctl create cluster --name bat13 --region <your-region>
```

> ⚠️ Ensure your AWS credentials (`aws configure`) are set before running this.

---

### 📌 Step 3: Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

---

### 📌 Step 4: Add Prometheus Helm Repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 📦 Install Prometheus & Grafana with Helm

```bash
kubectl create namespace prometheus

helm install kube-monitor prometheus-community/kube-prometheus-stack -n prometheus

# Optional workaround for timeout errors:
# helm install kube-monitor prometheus-community/kube-prometheus-stack -n prometheus --set prometheus.prometheusSpec.maximumStartupDurationSeconds=600
```

---

## 🌐 Expose Grafana & Prometheus Services

```bash
kubectl patch svc kube-monitor-kube-prometheus-sta-prometheus -n prometheus -p '{"spec": {"type": "LoadBalancer"}}'
kubectl patch svc kube-monitor-grafana -n prometheus -p '{"spec": {"type": "LoadBalancer"}}'
```

---

## 🔐 Access Grafana Dashboard

```bash
kubectl get secret --namespace prometheus kube-monitor-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

🖥️ Then, open:

* **Grafana:** `http://<LoadBalancer-URL>:80`
* **Prometheus:** `http://<LoadBalancer-URL>:9090`

---

## 🚀 Deploy Sample App: Facebook Clone

Here’s a sample Deployment & Service YAML used in this setup:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fb-deploy
  labels:
    app: facebook
    env: lab
spec:
  selector:
    matchLabels:
      app: facebook
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      name: fbpod
      labels:
        app: facebook
        env: lab
        Version: AC_V1
    spec:
      containers:
        - name: fbcont1
          image: devopshubg333/batch15d:9
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: fb-lb-svc
  labels:
    app: facebook
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: facebook
```

```bash
kubectl apply -f fb-app.yaml
```

---

## 📌 Observability Triangle

To troubleshoot efficiently, observe:

* **Metrics** – CPU, traffic, latency (via Prometheus)
* **Logs** – Application/system logs
* **Traces** – Request paths (add Jaeger for this)

---

## 📊 Dashboards You Should Monitor

| Category       | Key Metrics              |
| -------------- | ------------------------ |
| **Latency**    | Response time, timeouts  |
| **Traffic**    | Requests/sec, 5xx errors |
| **Errors**     | Error codes, HTTP errors |
| **Saturation** | CPU > 80%, memory        |

---

## 🙌 Contributing

Want to improve this stack or add Loki, Tempo, or AlertManager? Feel free to open a PR!

---

## ⭐ Star this repo if you found it helpful!

---


