# K8S Dashboard (UI)

Kubernetes Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage applications running in the cluster and troubleshoot them, as well as manage the cluster itself.
https://github.com/kubernetes/dashboard

## How to

1. Deploy dashboard

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml```

2. Create a user to access the dashboard `usr-admin`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: usr-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: usr-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: usr-admin
  namespace: kube-system
```

3. Apply the service account
```
kubectl apply -f eks-admin-service-account.yaml
```

4. Retrive token authentication
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep usr-admin | awk '{print $1}')
```

5. Start the kubectl proxy
```
kubectl proxy
```

6. Access the dashboard by the URL below and choose the TOKEN field.
```
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
```


