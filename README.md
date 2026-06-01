# 🚀 DevOps Project 004 — Kubernetes End-to-End Project on AWS EKS

<div align="center">

![AWS EKS](https://img.shields.io/badge/AWS-EKS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![kubectl](https://img.shields.io/badge/kubectl-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)

**Deploy and manage containerized 2048 Game on Amazon EKS with Load Balancing**

</div>

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
| **AWS EKS** | Managed Kubernetes cluster |
| **AWS CloudShell** | Browser-based CLI |
| **kubectl** | Kubernetes CLI |
| **Docker** | Container runtime |
| **AWS ELB** | Load Balancer for external access |
| **IAM** | Roles & permissions |

---

## 🚀 Implementation Steps

### STEP 1 — IAM Roles Banao

#### Role 1: eks-cluster-role
- Trusted entity: `AWS Service → EKS → EKS Cluster`
- Policy: `AmazonEKSClusterPolicy`

#### Role 2: eks-node-grp-role
- Trusted entity: `AWS Service → EC2`
- Policies:
  - `AmazonEKSWorkerNodePolicy`
  - `AmazonEC2ContainerRegistryReadOnly`
  - `AmazonEKS_CNI_Policy`

---

### STEP 2 — EKS Cluster Banao

**AWS Console → EKS → Create Cluster → Custom configuration**

| Setting | Value |
|---------|-------|
| Name | `eks-cluster-1` |
| Cluster IAM role | `eks-cluster-role` |
| Kubernetes version | 1.35 |
| VPC | Default VPC |
| Cluster endpoint access | Public and private |

> ⚠️ **Important:** "Custom configuration" select karo aur EKS Auto Mode **OFF** rakho!

⏳ 10-12 minute wait karo — Status ACTIVE hone ka

---

### STEP 3 — Node Group Add Karo

**EKS → eks-cluster-1 → Compute → Add node group**

| Setting | Value |
|---------|-------|
| Name | `eks-nodegrp-1` |
| Node IAM role | `eks-node-grp-role` |
| AMI type | Amazon Linux 2023 (x86_64) Standard |
| Instance type | `t3.medium` |
| Disk size | `20 GiB` |
| Desired/Min size | `1` |
| Maximum size | `2` |

⏳ 2-3 minute wait karo

---

### STEP 4 — kubectl Configure Karo

**AWS Console → CloudShell (>_ icon)**

```bash
# Identity verify karo
aws sts get-caller-identity

# kubeconfig update karo
aws eks update-kubeconfig --region us-east-1 --name eks-cluster-1

# Nodes check karo
kubectl get nodes
```

**Expected:**
```
NAME                           STATUS   ROLES    AGE   VERSION
ip-172-31-xx-xx.ec2.internal   Ready    <none>   2m    v1.35.x-eks-xxxxx
```

---

### STEP 5 — 2048 Game Pod Deploy Karo

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

> ⚠️ **Important Fix:** `blackicebird/2048` deprecated hai. Use: `public.ecr.aws/l6m2t8p7/docker-2048:latest`

---

### STEP 6 — LoadBalancer Service Banao

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

### STEP 7 — Game Access Karo

```bash
kubectl describe svc mygame-svc
```

Browser mein open karo:
```
http://<EXTERNAL-IP>
```

⏳ 2-3 minute wait karo DNS propagation ke liye — **2048 Game live! 🎮**

---

## 🐛 Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| `ErrImagePull` | Use `public.ecr.aws/l6m2t8p7/docker-2048:latest` |
| Node not Ready | Wait 2-3 min after node group creation |
| Site can't be reached | Wait 2-3 min for DNS propagation |
| EKS Auto Mode warnings | Select "Custom configuration", disable Auto Mode |

---

## 🧹 Cleanup

```bash
kubectl delete svc mygame-svc
kubectl delete pod 2048-pod
```

AWS Console: Node Group delete → Cluster delete

---

## 📚 What I Learned

- AWS EKS cluster setup aur configuration
- IAM Roles for EKS (Cluster + Node Group)
- kubectl commands for pod management
- Kubernetes Pod aur Service YAML
- LoadBalancer type service for external access
- AWS CloudShell for browser-based DevOps

---

## 👤 Author

**Hafiz Muhammad Umar Rafique**
- GitHub: [@hmurafique](https://github.com/hmurafique)

<div align="center">⭐ Star this repo if you found it helpful!</div>
