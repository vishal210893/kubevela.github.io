---
title: Debugging KubeVela Controllers
description: Learn how to debug KubeVela controllers using IDE tools
---

# Debugging KubeVela Controllers

This guide walks you through setting up a local debugging environment for KubeVela controllers. You'll learn how to configure your IDE (IntelliJ IDEA or GoLand) for effective debugging of the KubeVela core application.

## Overview

Debugging KubeVela controllers locally allows you to:
- Set breakpoints in controller code
- Step through code execution
- Inspect variables and application state
- Identify and fix issues more efficiently

## Prerequisites

Before you begin, ensure you have:

1. **Development Environment**
   - Go 1.19 or higher installed
   - IntelliJ IDEA (Ultimate) or GoLand
   - Git client
   - kubectl CLI tool

2. **Kubernetes Cluster**
   - A running Kubernetes cluster (local or remote)
   - Cluster admin access
   - kubectl configured to access your cluster

3. **Source Code**
   - KubeVela source code cloned locally
   ```bash
   git clone https://github.com/kubevela/kubevela.git
   cd kubevela
   ```

## Step 1: Install CRDs and Definitions

Before running the controller locally, you need to install the Custom Resource Definitions (CRDs) and system definitions in your cluster.

### Option A: Using Make (Recommended)

```bash
# Install CRDs
make install

# Install definitions
make def-install
```

### Option B: Manual Installation

```bash
# Install CRDs
kubectl apply -f charts/vela-core/crds/

# Install system definitions
kubectl apply -f charts/vela-core/templates/defwithtemplate/
```

### Verification

Verify the CRDs are installed:

```bash
kubectl get crds | grep oam.dev
```

You should see output listing KubeVela CRDs such as:
- `applications.core.oam.dev`
- `componentdefinitions.core.oam.dev`
- `traitdefinitions.core.oam.dev`
- etc.

## Step 2: Configure Your IDE

### IntelliJ IDEA / GoLand Setup

1. **Open the Project**
   - Launch IntelliJ IDEA or GoLand
   - Open the KubeVela repository directory

2. **Create a Run/Debug Configuration**
   - Go to **Run** → **Edit Configurations**
   - Click the **+** button and select **Go Build**
   - Configure the following settings:

   **Configuration Name**: `KubeVela Core`
   
   **Run kind**: `File`
   
   **Files**: Select `cmd/core/main.go`
   
   **Working directory**: Your KubeVela repository root
   
   **Environment variables** (optional):
   ```
   KUBECONFIG=/path/to/your/kubeconfig
   ```

3. **Apply and Save**
   - Click **Apply** and then **OK**

## Step 3: Start the Controller in Debug Mode

### Set Breakpoints

1. Navigate to the file you want to debug (e.g., a controller file)
2. Click in the gutter next to the line number where you want to pause execution
3. A red dot will appear indicating a breakpoint is set

### Start Debugging

1. Click the **Debug** icon (bug icon) in the toolbar, or
2. Press **Shift + F9** (macOS/Linux) or **Ctrl + Shift + F9** (Windows)
3. Select your **KubeVela Core** configuration

The controller will start and connect to your Kubernetes cluster. When code execution reaches a breakpoint, the IDE will pause and allow you to inspect the state.

## Step 4: Debugging Workflow

### Common Debugging Tasks

1. **Inspect Variables**
   - When paused at a breakpoint, hover over variables to see their values
   - Use the **Variables** panel to view all variables in scope

2. **Step Through Code**
   - **Step Over** (F8): Execute the current line and move to the next
   - **Step Into** (F7): Enter into function calls
   - **Step Out** (Shift + F8): Exit the current function

3. **Evaluate Expressions**
   - While paused, open the **Evaluate Expression** window (Alt + F8)
   - Enter Go expressions to evaluate in the current context

4. **Continue Execution**
   - Click **Resume Program** (F9) to continue until the next breakpoint

### Example: Debugging an Application Controller

1. Set a breakpoint in the Application controller's reconcile function:
   - File: `pkg/controller/core.oam.dev/v1beta1/application/application_controller.go`
   - Function: `Reconcile`

2. Apply a test application:
   ```bash
   kubectl apply -f examples/app-basic.yaml
   ```

3. The debugger will pause at your breakpoint when the controller processes the application

4. Inspect the `req` (reconcile request) and examine the application being processed

## Step 5: Working with Kubeconfig

### Using Custom Kubeconfig

If you need to use a specific kubeconfig file:

1. **Set Environment Variable in IDE**
   - Edit your run configuration
   - Add to **Environment variables**:
   ```
   KUBECONFIG=/path/to/your/kubeconfig
   ```

2. **Or Export in Terminal**
   ```bash
   export KUBECONFIG=/path/to/your/kubeconfig
   ```

### Multiple Cluster Contexts

To switch between different cluster contexts:

```bash
# List available contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>
```

## Troubleshooting

### Controller Won't Start

**Problem**: Controller fails to start with authentication errors

**Solution**: Verify your kubeconfig is correct and you have cluster access:
```bash
kubectl cluster-info
kubectl get nodes
```

### Breakpoints Not Hit

**Problem**: Code execution doesn't pause at breakpoints

**Solution**: 
- Ensure you're running in Debug mode, not Run mode
- Verify the breakpoint is on an executable line (not a comment or blank line)
- Check that the code path is actually being executed

### Port Already in Use

**Problem**: Error about port 9443 (webhook port) already in use

**Solution**: 
- Check if another instance is running: `ps aux | grep core`
- Kill the existing process or change the webhook port in the configuration

## Best Practices

1. **Start with CRDs**: Always ensure CRDs are installed before starting the controller
2. **Use Separate Test Cluster**: Debug against a test cluster, not production
3. **Clean State**: Restart the debugger if you make code changes
4. **Log Levels**: Increase log verbosity for more debugging information
5. **Test Resources**: Use simple test manifests to reproduce issues

## Next Steps

- For debugging with webhooks enabled, see [Debugging KubeVela with Webhook Integration](./debugging-kubevela-with-webhook.md)
- Learn about [Application Debugging](debug.md) for debugging deployed applications
- Explore [Dry Run](dry-run.md) for testing without actually applying changes

## Additional Resources

- [KubeVela Development Guide](../../contributor/code-contribute.md)
- [Controller Runtime Documentation](https://pkg.go.dev/sigs.k8s.io/controller-runtime)
- [Go Debugging in IntelliJ](https://www.jetbrains.com/help/idea/debugging-go-code.html)

