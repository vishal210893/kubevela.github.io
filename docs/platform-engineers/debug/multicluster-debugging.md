---
title: Multi-Cluster IDE Debugging
description: Learn how to set up a multicluster KubeVela environment for debugging the core controller from your IDE.
---

# KubeVela Core Multi-Cluster IDE Debugging Guide

This guide walks you through setting up a multicluster KubeVela environment for debugging the core controller from your IDE.

## Prerequisites

- k3d installed
- kubectl configured
- KubeVela source code
- IDE with debugging capabilities (e.g., GoLand, VS Code)

## Part 1: Master Cluster Setup

### Step 1: Create Master Cluster

```bash
# Command Format:
k3d cluster create {clustername}

# Example:
k3d cluster create master
```

### Step 2: Export Kubeconfig

```bash
# Command Format:
k3d kubeconfig get {clustername} > ~/.kube/{clustername}.yaml
export KUBECONFIG=~/.kube/{clustername}.yaml

# Example:
k3d kubeconfig get master > ~/.kube/master.yaml
export KUBECONFIG=~/.kube/master.yaml
```

### Step 3: Install Core Components

```bash
make core-install
make def-install
```

## Part 2: Configure Webhook Debugger

### Step 4: Update Debugger Script

Edit the `setup-webhook-debugger.sh` script and update the kubeconfig location:

```bash
#--- STEP 10: Export KUBECONFIG ------------------------------------------------
echo "==> STEP 10: Export KUBECONFIG"
export KUBECONFIG="${HOME}/.kube/master.yaml"
echo "Using KUBECONFIG=${KUBECONFIG}"
```

### Step 5: Run Debugger Setup

```bash
./setup-debugger.sh
```

## Part 3: IDE Configuration

### Step 6: Configure Environment Variables

In your IDE's build/debug configuration, add the following environment variable:

```bash
# Format:
KUBECONFIG={path-to-master-kubeconfig}

# Example:
KUBECONFIG=/Users/viskumar/.kube/master.yaml
```

Replace the path with the actual location of your master cluster's kubeconfig file.

**Why is this needed?**

When the core controller runs in your IDE, it needs to know which cluster to connect to. The KUBECONFIG environment variable tells the controller to use the master cluster's configuration.

### Step 7: Start the IDE in Debug Mode

Launch your IDE debugger. The core controller will now run from your IDE instead of in-cluster.

## Part 4: Install Cluster Gateway

### Understanding Cluster Gateway

**Why do we need Cluster Gateway?**

In a multicluster setup, Cluster Gateway is the critical component that enables:

- Communication between the master cluster and slave clusters
- Routing of API requests to remote clusters
- Centralised management of multiple Kubernetes clusters
- Secure authentication and authorisation across clusters

**Why run the core controller from the IDE while keeping the Cluster Gateway in-cluster?**

When debugging KubeVela core, we want to:

✅ Run the core controller from the IDE (so we can debug it with breakpoints)

✅ Keep Cluster Gateway running in-cluster (it needs to be accessible as a service endpoint for multicluster communication)

This is why we install only Cluster Gateway via Helm while the core controller runs in your IDE.

### Step 8: Prepare Helm Chart

Navigate to the Helm chart templates directory:

```bash
# Command Format:
cd {path-to-kubevela}/charts/vela-core/templates

# Example:
cd kubevela/charts/vela-core/templates
```

**Why delete other templates?**

The Vela-core Helm chart normally installs:

- Core controller deployment (we DON'T want this - it's running in our IDE)
- Cluster Gateway deployment (we NEED this for multicluster)
- Webhooks and other components (we DON'T need these for debugging)

By deleting everything except Cluster Gateway components, we ensure:

- No conflict between the in-cluster controller and the IDE controller
- Only the necessary multicluster infrastructure is deployed
- Clean separation of the debugging environment

**Files to KEEP (delete everything else):**

✅ `cluster-gateway/` (entire folder) - Cluster Gateway deployment and services

✅ `_helpers.tpl` - Helm template helpers required by other templates

✅ `kubevela-controller.yaml` - Core controller service account and RBAC (needs modification - see below)

✅ `addon_registry.yaml` - Addon registry configuration

✅ `NOTES.txt` - Post-installation notes

**Files/Folders to DELETE:**

❌ All webhook-related templates

❌ Application controller templates

❌ Any other deployment or pod templates

❌ Other service templates not related to Cluster Gateway

**⚠️ Important: Modify kubevela-controller.yaml**

Open `kubevela-controller.yaml` and remove the following definitions:

❌ **Deployment** - Delete the entire Deployment resource (the controller will run in your IDE)

❌ **Service** - Delete the Service resource (not needed when controller runs locally)

❌ **ServiceMonitor** - Delete ServiceMonitor resource if present (monitoring not needed for debugging)

**Keep ONLY the RBAC resources:**

✅ **ServiceAccount** - Keep this (controller needs identity)

✅ **ClusterRole** - Keep this (defines permissions)

✅ **ClusterRoleBinding** - Keep this (grants permissions to ServiceAccount)

✅ **Role** - Keep this if present

✅ **RoleBinding** - Keep this if present

After modification, `kubevela-controller.yaml` should contain ONLY RBAC-related Kubernetes resources.

:::warning
Only perform this cleanup AFTER running `make core-install` and `make def-install`. These commands install the CRDs and definitions needed by KubeVela.
:::

### Step 9: Install Cluster Gateway via Helm

```bash
helm install kubevela ./charts/vela-core \
  --set devLogs=true \
  --create-namespace \
  --namespace vela-system \
  --wait \
  --debug
```

### Step 10: Verify Installation

```bash
kubectl get all -n vela-system
```

Expected output:

```
NAME                                            READY   STATUS    RESTARTS   AGE
pod/kubevela-cluster-gateway-6f4785b888-9rzlx   1/1     Running   0          82s

NAME                                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/kubevela-cluster-gateway-service   ClusterIP   10.43.119.1   <none>        9443/TCP   82s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kubevela-cluster-gateway   1/1     1            1           82s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/kubevela-cluster-gateway-6f4785b888   1         1         1       82s
```

## Part 5: Slave Cluster Setup

### Step 11: Create Slave Cluster

```bash
# Command Format:
k3d cluster create {clustername}

# Example:
k3d cluster create slave
```

### Step 12: Export Slave Kubeconfig

```bash
# Command Format:
k3d kubeconfig get {clustername} > ~/.kube/{clustername}.yaml

# Example:
k3d kubeconfig get slave > ~/.kube/slave.yaml
```

## Part 6: Configure Kubeconfig for Multicluster

### Understanding the Configuration Changes

**Why do we need to modify kubeconfig files?**

By default, k3d clusters are configured to be accessed via localhost (0.0.0.0), which works fine for local development on a single machine. However, in a multicluster setup:

- **Host IP is required**: The master cluster needs to reach the slave cluster over the network, not via localhost
- **TLS verification skip**: k3d uses self-signed certificates that won't validate against the host IP
- **Certificate removal**: Since we're skipping TLS verification, we don't need the certificate authority data

These changes allow the Cluster Gateway to communicate with remote clusters using your host's actual network address.

### Step 13: Patch Kubeconfig Files

Navigate to your `.kube` directory (usually `~/.kube/`) and edit both `master.yaml` and `slave.yaml`.

**Required changes for BOTH files:**

1. Replace `server: https://0.0.0.0:{port}` with `server: https://{your-host-ip}:{port}`
2. Add `insecure-skip-tls-verify: true` under the cluster section
3. Remove the `certificate-authority-data` line

**How to find your host IP:**

```bash
# On Linux/Mac
hostname -I | awk '{print $1}'

# Or check your network settings
ifconfig | grep "inet " | grep -v 127.0.0.1
```

**Before (slave.yaml example):**

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
    server: https://0.0.0.0:49292
  name: k3d-slave
contexts:
- context:
    cluster: k3d-slave
    user: admin@k3d-slave
  name: k3d-slave
current-context: k3d-slave
kind: Config
preferences: {}
users:
- name: admin@k3d-slave
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
    client-key-data: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0t...
```

**After (slave.yaml example):**

```yaml
apiVersion: v1
clusters:
- cluster:
    server: https://10.131.101.169:49292
    insecure-skip-tls-verify: true
  name: k3d-slave
contexts:
- context:
    cluster: k3d-slave
    user: admin@k3d-slave
  name: k3d-slave
current-context: k3d-slave
kind: Config
preferences: {}
users:
- name: admin@k3d-slave
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t...
    client-key-data: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0t...
```

Repeat this process for `master.yaml` with the appropriate IP and port.

## Part 7: Join Clusters

### Step 14: Switch to Master Context

```bash
# Command Format:
export KUBECONFIG=~/.kube/{master-clustername}.yaml

# Example:
export KUBECONFIG=~/.kube/master.yaml
```

**Why switch to master context?**

The `vela cluster join` command needs to be executed from the master cluster context because:

- It registers the slave cluster with the master's Cluster Gateway
- It creates necessary secrets and configurations in the master cluster
- It establishes a trust relationship between clusters

### Step 15: Join Slave Cluster

```bash
# Command Format:
vela cluster join ~/.kube/{slave-clustername}.yaml

# Example:
vela cluster join ~/.kube/slave.yaml
```

**What happens during cluster join?**

This command:

- Reads the slave cluster's kubeconfig
- Creates a secret in the master cluster containing slave cluster credentials
- Registers the cluster with the Cluster Gateway
- Validates connectivity between clusters

### Step 16: Verify Cluster Registration

```bash
vela cluster ls
```

Expected output:

```
CLUSTER      ALIAS    TYPE               ENDPOINT                        ACCEPTED    LABELS
local                 Internal           -                               true
k3d-slave             X509Certificate    https://10.131.101.169:49292    true
```

## Part 8: Test Multicluster Deployment

### Step 17: Deploy Test Application

Apply the following application to verify multicluster functionality:

```yaml
apiVersion: core.oam.dev/v1beta1
kind: Application
metadata:
  name: app-deploy-io
spec:
  components:
    - name: podinfo
      outputs:
        - name: message
          valueFrom: output.status.conditions[0].message
        - name: ip
          valueFrom: outputs.service.spec.clusterIP
      properties:
        image: stefanprodan/podinfo:4.0.3
      type: webservice
      traits:
        - type: expose
          properties:
            port: [80]

    - name: configmap
      properties:
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: deployment-msg
      type: raw
      inputs:
        - from: message
          parameterKey: data.msg
        - from: ip
          parameterKey: data.ip

  policies:
    - name: topo
      properties:
        clusters: ["local", "k3d-slave"]
      type: topology
```

### Step 18: Verify Deployment

Check that pods are running in both clusters:

**Master cluster:**

```bash
# Command Format:
export KUBECONFIG=~/.kube/{master-clustername}.yaml
kubectl get pods

# Example:
export KUBECONFIG=~/.kube/master.yaml
kubectl get pods
```

**Slave cluster:**

```bash
# Command Format:
export KUBECONFIG=~/.kube/{slave-clustername}.yaml
kubectl get pods

# Example:
export KUBECONFIG=~/.kube/slave.yaml
kubectl get pods
```

You should see the podinfo pods and related resources running in both clusters, confirming successful multicluster deployment.

## Troubleshooting

### Issue: Controller can't connect to the cluster

- Verify the `KUBECONFIG` environment variable is set correctly in the IDE
- Check that `master.yaml` is accessible from the path specified

### Issue: Slave cluster not reachable

- Verify the host IP is correct in the kubeconfig files
- Ensure k3d clusters are running: `k3d cluster list`
- Check network connectivity between clusters

### Issue: Pods not deploying to the slave cluster

- Verify cluster is registered: `vela cluster ls`
- Check Cluster Gateway logs: `kubectl logs -n vela-system -l app=kubevela-cluster-gateway`
- Ensure topology policy includes the correct cluster names

