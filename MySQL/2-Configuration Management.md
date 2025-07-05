### MySQL on Kubernetes: Configuration Management 

#### 1. Environment Variables

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

#### 2. Custom Configurations (my.cnf)

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

**to verify.**
# How to Verify MySQL ConfigMap Mount and Resource Requests in Kubernetes

## 1. Verifying the ConfigMap Mount and Custom `my.cnf` Usage

### a. Confirm the ConfigMap Exists

Check that your ConfigMap was created and contains the expected data:

```bash
kubectl get configmap mysql-config -n mysql1
kubectl describe configmap mysql-config -n mysql1
```
- The output should show your `my.cnf` content under the `Data` section.

### b. Check Pod Volume Mounts

Inspect your MySQL pod to ensure the ConfigMap is mounted at the correct path:

```bash
kubectl get pods -n mysql1
kubectl describe pod  -n mysql1
```
- In the `Volumes` and `Volume Mounts` sections, look for `config-volume` mounted at `/etc/mysql/conf.d`[1][2][3].

### c. Verify the File Inside the Pod

Get a shell into the running MySQL pod and check the contents of the mounted file:

```bash
kubectl exec -it  -n mysql1 -- /bin/bash
# Inside the pod:
ls /etc/mysql/conf.d
cat /etc/mysql/conf.d/my.cnf
```
- The output should match your custom `my.cnf` content (e.g., `max_connections=200`).

### d. Confirm MySQL Loaded the Custom Configuration

Inside the MySQL pod, connect to MySQL and check the active configuration:

```bash
mysql -u root -p
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'sql_mode';
```
- The values should reflect those set in your custom `my.cnf` file[4][5][6].

## 2. Verifying Resource Requests and Limits

### a. Check Pod Resource Requests and Limits

Describe the MySQL pod to see the resource requests and limits:

```bash
kubectl describe pod  -n mysql1
```
- Look for the `Containers` section, which will show `Limits` and `Requests` for CPU and memory[7][8].

### b. List All Pods’ Resource Requests and Limits (Optional)

To see resource requests and limits for all pods in the namespace:

```bash
kubectl get pods -n mysql1 -o jsonpath="{range .items[*]}{.metadata.name}{':\t'}{range .spec.containers[*]}{.resources}{'\n'}{end}{end}"
```
- This will print the resource requests and limits for each pod.

## 3. Summary Table

| Task                                 | Command Example                                                      | What to Check For                                 |
|---------------------------------------|---------------------------------------------------------------------|---------------------------------------------------|
| Check ConfigMap exists                | `kubectl get/describe configmap`                                    | `my.cnf` present in data                          |
| Check pod volume mounts               | `kubectl describe pod `                                        | `config-volume` at `/etc/mysql/conf.d`            |
| Inspect file in pod                   | `kubectl exec -it  -- ls/cat /etc/mysql/conf.d/my.cnf`         | File exists and content matches your config        |
| Confirm MySQL config loaded           | `SHOW VARIABLES LIKE ...` in MySQL shell                            | Values match your custom `my.cnf`                 |
| Check resource requests and limits    | `kubectl describe pod `                                        | CPU/memory requests and limits as specified       |

## 4. Best Practices

- Always verify both the presence of the file and that MySQL is using the intended configuration.
- Monitor pod resource usage to ensure requests and limits are appropriate for your workload.
- If you update the ConfigMap, restart the pod to ensure the new configuration is loaded[9][10].

By following these steps and using the provided commands, you can confidently verify that your custom MySQL configuration and resource management are correctly applied in your Kubernetes deployment.

[1] https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
[2] https://kubernetes.io/docs/concepts/configuration/configmap/
[3] https://www.kubermatic.com/blog/keeping-the-state-of-apps-part-3-introduction-to-configmaps/
[4] https://dba.stackexchange.com/questions/703/confirm-that-my-cnf-file-has-loaded-ok
[5] https://stackoverflow.com/questions/580331/determine-which-mysql-configuration-file-is-being-used
[6] https://github.com/bitnami/charts/issues/5817
[7] https://spot.io/resources/kubernetes-architecture/kubernetes-limits-vs-requests-key-differences-and-how-they-work/
[8] https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
[9] https://stackoverflow.com/questions/75671855/how-to-update-a-config-map-in-kubernetes
[10] https://spacelift.io/blog/kubernetes-configmap
[11] https://stackoverflow.com/questions/56871647/how-to-fetch-configmap-from-kubernetes-pod
[12] https://labex.io/tutorials/kubernetes-how-to-verify-configmap-data-in-a-running-kubernetes-application-415774
[13] https://www.qovery.com/blog/kubernetes-configmap-our-complete-guide/
[14] https://kubernetes.io/docs/concepts/policy/resource-quotas/
[15] https://www.reddit.com/r/openshift/comments/w8pg7w/unable_to_edit_mycnf_file_inside_mysql_pod/
[16] https://www.aquasec.com/cloud-native-academy/kubernetes-101/kubernetes-configmap/
[17] https://sysdig.com/blog/kubernetes-limits-requests/
[18] https://serverfault.com/questions/1106547/check-if-config-map-or-secret-is-used
[19] https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/
[20] https://labex.io/tutorials/kubernetes-how-to-resolve-configmap-mounting-issues-418604
[21] https://stackoverflow.com/questions/66007364/how-can-i-summarize-all-the-resource-limits-and-requests-in-kubernetes-with-kube
[22] https://serverfault.com/questions/914031/how-to-provide-mysql-my-cnf-after-s2i-has-run
[23] https://gist.github.com/lucasrenan/c686e204ed7fb8e52825de34bc437771
[24] https://stackoverflow.com/questions/74847314/how-to-edit-my-cnf-file-which-is-present-in-kubernetes-mysql-container
[25] https://serverfault.com/questions/1118822/should-i-add-a-mysql-uri-to-the-mysql-kubernetes-deployment-file
[26] https://kubernetes.io/docs/reference/kubectl/generated/kubectl_exec/
[27] https://techdocs.broadcom.com/us/en/vmware-tanzu/data-solutions/tanzu-for-mysql-on-kubernetes/1-10/vmware-mysql-k8s/accessing.html
[28] https://kubedb.com/docs/v2025.5.30/reference/cli/kubectl-dba_exec_mysql/
[29] https://dev.mysql.com/doc/mysql-operator/en/mysql-operator-logs.html
[30] https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/
[31] https://portworx.com/knowledge-hub/mysql-on-kubernetes/
[32] https://stackoverflow.com/questions/48150179/how-to-mount-entire-directory-in-kubernetes-using-configmap


