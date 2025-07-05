### Kubernetes with MySQL DBA on Linux
#### [Kubernetes Architecutre]([https://github.com/venkat-mr369/k8s-for-DBAs/GKE-Notes.md](https://github.com/venkat-mr369/k8s-for-DBAs/blob/main/GKE-Notes.md))
## Overview

Running MySQL on Kubernetes in a Linux environment combines the flexibility of container orchestration with the reliability of Linux. A MySQL DBA (Database Administrator) in this setup is responsible for both traditional database management and Kubernetes-native operations.

## Core DBA Activities for MySQL on Kubernetes

### 1. Provisioning and Deployment

- **Cluster Preparation**
  - Ensure the Kubernetes cluster is set up and accessible.
  - Prepare nodes with sufficient CPU, memory, and storage for database workloads.
- **MySQL Deployment**
  - Use Kubernetes manifests (YAML files) to define MySQL deployments, StatefulSets, Persistent Volumes (PVs), and Persistent Volume Claims (PVCs).
  - Example steps:
    - Create a Kubernetes Secret for the MySQL root password.
    - Define a PVC for persistent storage.
    - Deploy MySQL using a Deployment or StatefulSet for high availability.

### 2. Configuration Management

- **Environment Variables**
  - Set MySQL configuration parameters (e.g., root password, database name) via environment variables in deployment YAML.
- **Custom Configurations**
  - Mount custom `my.cnf` files using ConfigMaps or volumes for advanced tuning.
- **Resource Requests and Limits**
  - Define CPU and memory requests/limits to ensure MySQL pods have adequate resources.

### 3. Storage and Data Management

- **Persistent Storage**
  - Use PVs and PVCs to ensure MySQL data persists across pod restarts and rescheduling.
  - Choose storage classes optimized for database workloads (e.g., SSD-backed).
- **Volume Expansion**
  - Expand storage as data grows by updating PVCs and underlying storage.

### 4. High Availability and Scalability

- **StatefulSets**
  - Use StatefulSets for multi-node MySQL clusters, providing stable network identities and persistent storage for each replica.
- **Replication**
  - Configure MySQL Group Replication or use Operators to manage primary-replica setups for high availability.
- **Pod Affinity/Anti-Affinity**
  - Distribute MySQL pods across nodes to enhance resilience.

### 5. Security Management

- **Secrets Management**
  - Store sensitive data (passwords, keys) in Kubernetes Secrets.
- **Network Policies**
  - Restrict access to MySQL pods using Kubernetes Network Policies.
- **RBAC**
  - Use Role-Based Access Control to limit who can manage MySQL resources.
- **Encryption**
  - Enable SSL/TLS for data in transit and use storage-level encryption for data at rest.

### 6. Backup and Restore

- **Automated Backups**
  - Schedule regular backups using Kubernetes-native tools, Operators, or third-party solutions.
  - Store backups in external storage (e.g., S3, Azure Blob).
- **Restore Operations**
  - Restore databases from backups using Operator CRDs or manual scripts.
  - Test restores regularly to ensure data integrity.

### 7. Monitoring and Performance Tuning

- **Resource Monitoring**
  - Use tools like Prometheus, Grafana, or Kubernetes Metrics Server to monitor CPU, memory, disk, and MySQL-specific metrics.
- **Query Optimization**
  - Analyze slow queries, optimize indexes, and tune MySQL parameters.
- **Alerting**
  - Set up alerts for resource exhaustion, replication lag, or failed backups.

### 8. Upgrades and Patching

- **Rolling Updates**
  - Use Kubernetes rolling update strategies for MySQL pods, ensuring minimal downtime.
- **Operator-Driven Upgrades**
  - Leverage MySQL Operators to automate version upgrades and patching.
- **Kubernetes Upgrades**
  - Coordinate MySQL upgrades with Kubernetes cluster upgrades for compatibility.

### 9. Disaster Recovery and Failover

- **Replication and Failover**
  - Set up automated failover using MySQL Group Replication or Operators.
- **Cross-Cluster Replication**
  - Replicate data across clusters for disaster recovery.
- **Disaster Recovery Drills**
  - Regularly test failover and recovery procedures.

### 10. Day-to-Day Maintenance

- **User and Permission Management**
  - Manage MySQL users and privileges using SQL or Kubernetes Secrets.
- **Log Management**
  - Collect and analyze MySQL logs using Kubernetes logging solutions.
- **Routine Health Checks**
  - Monitor database health, check replication status, and review storage usage.

## Example: MySQL Deployment YAML (Simplified)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 3
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
        image: mysql:8.0
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
```

## Typical MySQL DBA Daily Checklist on Linux

| Activity              | Description                                                        |
|-----------------------|--------------------------------------------------------------------|
| Backup & Restore      | Schedule, monitor, and test backups; restore as needed             |
| User Management       | Add, remove, and update user privileges                            |
| Performance Tuning    | Monitor slow queries, optimize indexes, tune configs               |
| Storage Management    | Monitor disk usage, expand volumes, manage PVCs                    |
| Security              | Rotate passwords, manage secrets, enforce network policies         |
| Monitoring & Alerting | Set up and review alerts for resource usage and failures           |
| Upgrades & Patching   | Apply MySQL and Kubernetes updates, test in staging                |
| Disaster Recovery     | Test failover, cross-cluster replication, and restore procedures   |
| Log Management        | Collect and analyze logs for troubleshooting                       |
| Documentation         | Maintain runbooks, backup logs, and incident reports               |

## Best Practices

- Use Operators for lifecycle automation (provisioning, scaling, backup, restore, upgrades).
- Always use persistent storage for MySQL data.
- Isolate MySQL workloads using namespaces and network policies.
- Regularly test backup and restore procedures.
- Monitor both Kubernetes and MySQL metrics for holistic health checks.
- Document all changes and maintain clear runbooks for incident response.
