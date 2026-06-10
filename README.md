# 🚀 DevOps Project 004 
# Kubernetes End-to-End Project on AWS EKS

![AWS EKS](https://img.shields.io/badge/AWS%20EKS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

> Deploy and manage a containerized 2048 Game on Amazon EKS with Load Balancing

---

## 📋 Architecture Overview

```
AWS Console
    ↓
IAM Roles (eks-cluster-role + eks-node-grp-role)
    ↓
EKS Cluster (Control Plane)
    ↓
Node Group (EC2 t3.medium Worker Node)
    ↓
kubectl (AWS CloudShell)
    ↓
Pod (2048 Game Container)
    ↓
LoadBalancer Service (ELB)
    ↓
Internet → Browser → 🎮 2048 Game!
```

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| ![AWS EKS](https://img.shields.io/badge/AWS%20EKS-FF9900?style=flat&logo=amazon-aws&logoColor=white) | Managed Kubernetes Cluster |
| ![CloudShell](https://img.shields.io/badge/AWS%20CloudShell-FF9900?style=flat&logo=amazon-aws&logoColor=white) | Browser-Based CLI |
| ![kubectl](https://img.shields.io/badge/kubectl-326CE5?style=flat&logo=kubernetes&logoColor=white) | Kubernetes CLI |
| ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white) | Container Runtime |
| ![ELB](https://img.shields.io/badge/AWS%20ELB-FF9900?style=flat&logo=amazon-aws&logoColor=white) | Load Balancer for external access |
| ![IAM](https://img.shields.io/badge/AWS%20IAM-FF9900?style=flat&logo=amazon-aws&logoColor=white) | Roles & Permissions |

---

## 🚀 Implementation Steps

### STEP 1 — Create IAM Roles

**Role 1: `eks-cluster-role`**
- Trusted entity: AWS Service → EKS → EKS Cluster
- Policy: `AmazonEKSClusterPolicy`

**Role 2: `eks-node-grp-role`**
- Trusted entity: AWS Service → EC2
- Policies:
  - `AmazonEKSWorkerNodePolicy`
  - `AmazonEC2ContainerRegistryReadOnly`
  - `AmazonEKS_CNI_Policy`

---

### STEP 2 — Create the EKS Cluster

Navigate to: **AWS Console → EKS → Create Cluster → Custom Configuration**

| Setting | Value |
|---------|-------|
| Name | `eks-cluster-1` |
| Cluster IAM role | `eks-cluster-role` |
| Kubernetes version | `1.35` |
| VPC | Default VPC |
| Cluster endpoint access | Public and Private |

> ⚠️ **Important:** Select **"Custom Configuration"** and ensure **EKS Auto Mode is OFF**.

> ⏳ Wait 10–12 minutes for the cluster status to become **ACTIVE**.

---

### STEP 3 — Add a Node Group

Navigate to: **EKS → eks-cluster-1 → Compute → Add Node Group**

| Setting | Value |
|---------|-------|
| Name | `eks-nodegrp-1` |
| Node IAM role | `eks-node-grp-role` |
| AMI type | Amazon Linux 2023 (x86_64) Standard |
| Instance type | `t3.medium` |
| Disk size | 20 GiB |
| Desired / Min size | 1 |
| Maximum size | 2 |

> ⏳ Wait 2–3 minutes for the node group to become ready.

---

### STEP 4 — Configure kubectl

Open **AWS CloudShell** from the console toolbar (`>_` icon).

```bash
# Verify your identity
aws sts get-caller-identity

# Update kubeconfig for your cluster
aws eks update-kubeconfig --region us-east-1 --name eks-cluster-1

# Confirm Nodes are ready
kubectl get nodes
```

**Expected Output:**
```
NAME                           STATUS   ROLES    AGE   VERSION
ip-172-31-xx-xx.ec2.internal   Ready    <none>   2m    v1.35.x-eks-xxxxx
```

---

### STEP 5 — Deploy the 2048 Game Pod

```bash
cat > 2048-pod.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
   name: 2048-pod
   labels:
      app: 2048-ws
spec:
   containers:
   - name: 2048-container
     image: public.ecr.aws/l6m2t8p7/docker-2048:latest
     ports:
       - containerPort: 80
EOF

kubectl apply -f 2048-pod.yaml
kubectl get pods
```

> ⚠️ **Note:** The `blackicebird/2048` image is deprecated. Use `public.ecr.aws/l6m2t8p7/docker-2048:latest` instead.

---

### STEP 6 — Create the LoadBalancer Service

```bash
cat > mygame-svc.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
   name: mygame-svc
spec:
   selector:
      app: 2048-ws
   ports:
   - protocol: TCP
     port: 80
     targetPort: 80
   type: LoadBalancer
EOF

kubectl apply -f mygame-svc.yaml
kubectl get svc mygame-svc
```

---

### STEP 7 — Access the Game

```bash
kubectl describe svc mygame-svc
```

Open your browser and navigate to:

```
http://<EXTERNAL-IP>
```

> ⏳ Wait 2–3 minutes for DNS propagation — the 2048 Game is now live! 🎮

---

## 🐛 Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| `ErrImagePull` | Use `public.ecr.aws/l6m2t8p7/docker-2048:latest` |
| Node not Ready | Wait 2–3 minutes after node group creation |
| Site can't be reached | Wait 2–3 minutes for DNS propagation |
| EKS Auto Mode warnings | Select "Custom configuration" and disable Auto Mode |

---

## 🧹 Cleanup

```bash
kubectl delete svc mygame-svc
kubectl delete pod 2048-pod
```

Then in the AWS Console: **Delete Node Group → Delete Cluster**

---

## 📚 What I Learned

- AWS EKS cluster setup and configuration
- IAM Roles for EKS (Cluster & Node Group)
- `kubectl` commands for pod management
- Kubernetes Pod and Service YAML manifests
- LoadBalancer service type for external access
- AWS CloudShell for browser-based DevOps workflows

---

## 👤 Author

**Hafiz Muhammad Umar Rafique**
GitHub: [@hmurafique](https://github.com/hmurafique)

---

> ⭐ Star this repo if you found it helpful!
