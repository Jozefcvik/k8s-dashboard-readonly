# Kubernetes Dashboard – Read-Only Access (RBAC + Ingress)

This repository provides a secure **read-only configuration** for the 
[Kubernetes Dashboard](https://github.com/kubernetes/dashboard) using:

- ✅ Dedicated `ServiceAccount`
- ✅ `ClusterRoleBinding` with `view` permissions
- ✅ Secure HTTPS Ingress (NGINX)
- ✅ Optional IP whitelisting

The goal is to expose the dashboard safely with **read-only permissions** for monitoring and inspection purposes.

---

### 📦 Included Manifests

#### 1️⃣ `dashboard-k8s-readonly.yaml`

Creates:

- `ServiceAccount` → `dashboard-readonly`
- `ClusterRoleBinding` → binds to built-in `view` ClusterRole

##### This ensures the user can:

- View pods, deployments, services, configmaps, etc.
- Cannot modify, delete, or create resources.
- Cannot access secrets (default `view` role behavior).

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-readonly
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-readonly
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
- kind: ServiceAccount
  name: dashboard-readonly
  namespace: kubernetes-dashboard
```

#### 2️⃣ ingress-dashboard.yaml

Creates a secure HTTPS Ingress for the Kubernetes Dashboard.

##### Features:

- Uses NGINX Ingress Controller
- Forces SSL redirect
- Uses HTTPS backend protocol
- Optional IP whitelist for additional security
- Custom hostname

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/whitelist-source-range: "63.178.174.163"
spec:
  ingressClassName: nginx
  rules:
  - host: dashboard.jozefcvik.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
```

---

### 🚀 Installation

#### ✅ Make sure:

- Kubernetes Dashboard is already deployed
- NGINX Ingress Controller is installed
- DNS is properly configured for your host

### 📦 Apply manifests:
```bash
kubectl apply -f dashboard-k8s-readonly.yaml
kubectl apply -f ingress-dashboard.yaml
```

---

### 🔐 Accessing the Dashboard

#### 1️⃣ Retrieve token for the read-only ServiceAccount:
```bash
kubectl -n kubernetes-dashboard create token dashboard-readonly
```

#### 2️⃣ Open your browser:
```code
https://dashboard.jozefcvik.com
```

#### 3️⃣ Login using the generated token.

