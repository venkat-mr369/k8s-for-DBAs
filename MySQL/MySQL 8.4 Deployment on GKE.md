#### MySQL 8.4 Deployment on GKE (Kubernetes) – Step-by-Step Guide

Your GKE cluster (`my-gke-cluster`) is running in `asia-east1-a` with two `e2-medium` nodes and Kubernetes version 1.32.4-gke.1415000. Below is a secure, modern workflow for deploying MySQL 8.4 using Kubernetes manifests, following best practices and avoiding deprecated features.

#### 1. Create a Namespace (Optional)

Isolate your MySQL resources:

```bash
kubectl create namespace mysql1
```

#### 2. Create a Kubernetes Secret for MySQL Root Password

Store the MySQL root password securely:

```bash
kubectl create secret generic mysql-secret \
  --from-literal=password='MySQL@123' \
  --namespace=mysql1
```
*Replace `'YourRootPassword'` with a strong password.*

## 3. Define a Persistent Volume Claim (PVC) for MySQL

Create a PVC to ensure data persists across pod restarts:

```yaml
# mysql-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: mysql1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

Apply the PVC:

```bash
kubectl apply -f mysql-pvc.yaml
```

## 4. Deploy MySQL 8.4 Using a StatefulSet

A StatefulSet ensures stable network identity and persistent storage for each MySQL pod.

```yaml
# mysql-statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: mysql1
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.4
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
      storageClassName: standard
```

Apply the StatefulSet:

```bash
kubectl apply -f mysql-statefulset.yaml
```

## 5. Expose MySQL with a Headless Service

A headless service is recommended for StatefulSets:

```yaml
# mysql-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: mysql1
spec:
  ports:
  - port: 3306
    name: mysql
  clusterIP: None
  selector:
    app: mysql
```

Apply the service:

```bash
kubectl apply -f mysql-service.yaml
```

## 6. Verification

Check the status of your deployment:

```bash
kubectl get pods -n mysql1
kubectl get pvc -n mysql1
kubectl get svc -n mysql1
```

All resources should show as `Running` or `Bound`.

## 7. Best Practices

- **Do not use deprecated Kubelet readonly port (10255).** All modern monitoring and management should use secure endpoints or the Kubernetes API.
- **Use secrets for all sensitive data.**
- **Use persistent storage for all database data.**
- **Scale replicas as needed for high availability.**
- **Monitor your cluster and database using Stackdriver or other supported tools.**

## Summary Table

| Step                | Command/YAML Example                | Purpose                                 |
|---------------------|-------------------------------------|-----------------------------------------|
| Create Namespace    | `kubectl create namespace mysql1`   | Isolate MySQL resources                 |
| Create Secret       | `kubectl create secret generic ...` | Store MySQL root password securely      |
| Create PVC          | `mysql-pvc.yaml`                    | Persistent storage for MySQL            |
| Deploy StatefulSet  | `mysql-statefulset.yaml`            | Deploy MySQL 8.4 with persistence       |
| Expose Service      | `mysql-service.yaml`                | Network access to MySQL pods            |
| Verify Resources    | `kubectl get ...`                   | Ensure all resources are running        |

This workflow ensures a secure, production-ready MySQL 8.4 deployment on your GKE cluster, fully aligned with current Kubernetes and GCP best practices.
