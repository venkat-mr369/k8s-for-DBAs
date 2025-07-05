### MySQL on Kubernetes: Configuration Management 

## 1. Environment Variables

**Purpose:**  
Environment variables are used in Kubernetes deployment YAML files to pass configuration parameters to the MySQL container securely and flexibly.

**Key Points:**
- Common parameters include the root password and the default database name.
- Sensitive values (like passwords) should be stored in Kubernetes Secrets and referenced in the YAML.

**Example: Setting MySQL Root Password and Database Name**
```yaml
env:
- name: MYSQL_ROOT_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: password
- name: MYSQL_DATABASE
  value: myappdb
```
- In this example, the root password is securely stored in a Secret (`mysql-secret`) with the value `MySQL@123`, and the default database is set to `myappdb`.

**How to Create the Secret:**
```bash
kubectl create secret generic mysql-secret \
  --from-literal=password='MySQL@123' \
  --namespace=mysql1
```

## 2. Custom Configurations (my.cnf)

**Purpose:**  
Custom MySQL configuration (e.g., tuning parameters) can be provided by mounting a custom `my.cnf` file into the container using a ConfigMap.

**Steps:**
1. **Create a custom `my.cnf` file** (example content):
    ```ini
    [mysqld]
    max_connections=200
    sql_mode=STRICT_ALL_TABLES
    ```
2. **Create a ConfigMap from the file:**
    ```bash
    kubectl create configmap mysql-config \
      --from-file=my.cnf=./my.cnf \
      --namespace=mysql1
    ```
3. **Mount the ConfigMap in the deployment YAML:**
    ```yaml
    volumeMounts:
    - name: mysql-persistent-storage
      mountPath: /var/lib/mysql
    - name: config-volume
      mountPath: /etc/mysql/conf.d

    volumes:
    - name: config-volume
      configMap:
        name: mysql-config
    ```
- This ensures MySQL uses your custom configuration at startup.

## 3. Resource Requests and Limits

**Purpose:**  
Resource requests and limits ensure that MySQL pods have enough CPU and memory to run efficiently, and prevent them from consuming excessive resources that could impact other workloads.

**How to Define in YAML:**
```yaml
resources:
  requests:
    memory: "1Gi"
    cpu: "500m"
  limits:
    memory: "2Gi"
    cpu: "1"
```
- **Requests**: Minimum resources guaranteed for the pod.
- **Limits**: Maximum resources the pod can use.

**Placement:**  
Add the `resources` section under the container spec in your StatefulSet or Deployment YAML.

## Summary Table

| Topic                      | Implementation Example / Command                                                                 | Purpose                                      |
|----------------------------|-----------------------------------------------------------------------------------------------|----------------------------------------------|
| Environment Variables      | `env:` in YAML, use `kubectl create secret`                                                   | Pass MySQL config (root password, db name)   |
| Custom Configurations      | `kubectl create configmap`, mount in YAML                                                     | Advanced MySQL tuning via custom `my.cnf`    |
| Resource Requests & Limits | `resources:` in YAML                                                                          | Control CPU/memory for MySQL pods            |

**By following these practices, you ensure secure, tunable, and resource-efficient MySQL deployments on Kubernetes, fully aligned with modern best practices.**
