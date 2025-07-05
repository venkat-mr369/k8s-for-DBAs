#### GKE Cluster Creation Using gcloud 

### 1. Cluster Preparation

### a. Create a Minimal GKE Cluster

Use the following command to create a small, single-zone GKE cluster with a specific node size and count:

```bash
gcloud container clusters create my-gke-cluster \
  --zone asia-east1-a \
  --num-nodes=2 \
  --machine-type=e2-medium
```
- `my-gke-cluster`: Name of your cluster.
- `--zone us-central1-a`: Choose your preferred zone.
- `--num-nodes=2`: Number of nodes (adjust as needed).
- `--machine-type=e2-medium`: Node size (2 vCPU, 4 GB RAM per node; adjust for your workload).
#### output
```bash
Note: The Kubelet readonly port (10255) is now deprecated. Please update your workloads to use the recommended alternatives. See https://cloud.google.com/kubernetes-engine/docs/how-to/disable-kubelet-readonly-port for ways to check usage and for migration instructions.
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster my-gke-cluster in asia-east1-a... Cluster is being health-checked (Kubernetes Control Plane is healthy)...done.                                                                                         
Created [https://container.googleapis.com/v1/projects/splendid-sled-460802-q9/zones/asia-east1-a/clusters/my-gke-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/asia-east1-a/my-gke-cluster?project=splendid-sled-460802-q9
kubeconfig entry generated for my-gke-cluster.
NAME: my-gke-cluster
LOCATION: asia-east1-a
MASTER_VERSION: 1.32.4-gke.1415000
MASTER_IP: 35.201.252.235
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.32.4-gke.1415000
NUM_NODES: 2
STATUS: RUNNING
```

### b. Get Cluster Credentials

After creation, configure `kubectl` to access your cluster:

```bash
gcloud container clusters get-credentials my-gke-cluster --zone asia-east1-a
```

### c. Verify Cluster and Node Status

Check that your cluster and nodes are ready:

```bash
kubectl get nodes
```
- All nodes should show a `Ready` status[1][4].

## 2. Node Sizing Guidance

| Machine Type   | vCPU | Memory (GB) | Use Case Example                |
|----------------|------|-------------|---------------------------------|
| e2-medium      | 2    | 4           | Small dev/test, light workloads |
| e2-standard-4  | 4    | 16          | Medium DB, web apps             |
| n2-standard-8  | 8    | 32          | Heavier DB, production          |

- Choose node size and count based on your database workload requirements[3][5].

## 3. Summary Table

| Step                | Command Example                                         | Purpose                        |
|---------------------|--------------------------------------------------------|--------------------------------|
| Create Cluster      | `gcloud container clusters create ...`                  | Provision GKE cluster          |
| Get Credentials     | `gcloud container clusters get-credentials ...`         | Access cluster with kubectl    |
| Verify Nodes        | `kubectl get nodes`                                     | Ensure nodes are ready         |

This setup ensures your Kubernetes cluster is ready and accessible, with nodes sized appropriately for database workloads.

