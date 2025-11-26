# Getting Started

This guide will help you get started with Polyfea using Helm charts - the easiest and recommended way to deploy Polyfea to Kubernetes.

## Prerequisites

Before you begin, ensure you have:

- **Kubernetes cluster** (v1.19+) - Use [Kind](https://kind.sigs.k8s.io/), [Minikube](https://minikube.sigs.k8s.io/), or any cloud provider
- **kubectl** - Configured to access your cluster
- **Helm** (v3.0+) - Package manager for Kubernetes

### Verify Your Setup

Check that your tools are installed and configured:

```bash
# Check Kubernetes connection
kubectl cluster-info

# Check Helm version
helm version

# Verify you can create namespaces
kubectl auth can-i create namespaces
```

## Quick Start with Helm

### Step 1: Add the Polyfea Helm Repository

```bash
helm repo add polyfea https://polyfea.github.io/charts
helm repo update
```

### Step 2: Install Polyfea Controller

Install the controller in its own namespace:

```bash
helm install polyfea-controller polyfea/polyfea-controller \
  --namespace polyfea-system \
  --create-namespace
```

Verify the installation:

```bash
# Check the controller pod is running
kubectl get pods -n polyfea-system

# Check the CRDs were installed
kubectl get crds | grep polyfea
```

You should see the following CRDs:
- `microfrontendclasses.polyfea.github.io`
- `microfrontends.polyfea.github.io`
- `webcomponents.polyfea.github.io`

### Step 3: Install Sample Applications (Optional)

Try the Material Design Shell samples to see Polyfea in action:

```bash
# If you did not execute the previous command, run this to install both controller and samples  
helm install polyfea-samples polyfea/polyfea-md-shell-samples \
  --namespace polyfea \
  --create-namespace

# If you already have the controller installed, just install the samples
helm install polyfea-samples polyfea/polyfea-md-shell-samples \
  --namespace polyfea \
  --create-namespace \
  --set polyfea-controller.enabled=false
```

This installs:
- The Polyfea controller (as a dependency)
- Sample microfrontend applications
- Example MicroFrontendClass, MicroFrontend, and WebComponent resources

### Step 4: Access the Application

Port-forward to access the sample application:

```bash
# Find the service name (namespace may vary)
kubectl get services -n polyfea-system

# Port-forward to the service
kubectl port-forward -n polyfea-system svc/<service-name> 8080:80
```

Open your browser to `http://localhost:8080/fea`

## Customizing Your Installation

### Configure Sample Applications

Disable specific samples:

```bash
helm install polyfea-samples polyfea/polyfea-md-shell-samples \
  --set samples.earthSample=false \
  --namespace polyfea \
  --create-namespace
```

### Custom Values

Create a `values.yaml` file to customize your installation:

```yaml
# values.yaml
controller:
  replicas: 2
  resources:
    limits:
      memory: "512Mi"
      cpu: "500m"
    requests:
      memory: "256Mi"
      cpu: "100m"

# Add custom annotations
annotations:
  example.com/team: "platform"
```

Install with custom values:

```bash
helm install polyfea-controller polyfea/polyfea-controller \
  --namespace polyfea-system \
  --create-namespace \
  --values values.yaml
```

## Creating Your First MicroFrontend

### 1. Define a MicroFrontendClass

In this section we will showcase a minimal use case where we will insert 
a native html element into the shell context of our application.

Create a file `my-frontend-class.yaml`:

```yaml
apiVersion: polyfea.github.io/v1alpha1
kind: MicroFrontendClass
metadata:
  name: my-app
  namespace: polyfea
spec:
  baseUri: "/app"
  title: "My Application"
  cspHeader: "default-src 'self'; script-src 'self' 'unsafe-inline'; frame-src 'self' https://polyfea.github.io; style-src 'self' 'unsafe-inline'"
```

Apply it:

```bash
kubectl apply -f my-frontend-class.yaml
```

### 2. Create WebComponents

We define a simple web component consisting of native html element so no MicroFrontend is needed.
Define which components to load:

```yaml
apiVersion: polyfea.github.io/v1alpha1
kind: WebComponent
metadata:
  name: my-iframe
  namespace: polyfea
spec:
  element: iframe
  attributes:
    - name: title
      value: "Inline Frame Example"
    - name: width
      value: "300"
    - name: height
      value: "200"
    - name: src
      value: "https://polyfea.github.io/charts/"
  displayRules:
    - anyOf:
      - context-name: shell
```

This will show a small iframe in the shell context area. With our chart repository as source.

Apply it:

```bash
kubectl apply -f my-iframe.yaml
```

### 4. Verify Your Resources

Check the status of your resources:

```bash
# Check MicroFrontendClass
kubectl get microfrontendclasses -n polyfea

# Check WebComponents
kubectl get webcomponents -n polyfea
```

Look for the `Ready` condition in the status to confirm everything is working.

Navigate to your application at `http://localhost:8080/app` to see your MicroFrontend in action!

## Next Steps

Now that you have Polyfea running:

1. **Explore the Samples** - Examine the sample resources to understand best practices
2. **Read the Architecture Guide** - Learn about context areas and display rules
3. **Browser Integration** - Set up the [@polyfea/core](https://www.npmjs.com/package/@polyfea/core) library
4. **Configure Namespace Policies** - Set up multi-tenancy for your teams
5. **Enable PWA Support** - Add Progressive Web App capabilities

## Troubleshooting

### Controller Not Starting

Check the logs:

```bash
kubectl logs -n polyfea-system -l app.kubernetes.io/name=polyfea-controller
```

### MicroFrontend Not Ready

Check the status:

```bash
kubectl describe microfrontend <name> -n <namespace>
```

Common issues:
- **Service not found** - Verify the service exists in the specified namespace
- **Namespace policy** - Check if the MicroFrontendClass allows your namespace
- **FrontendClass not found** - Ensure the MicroFrontendClass exists

### Resources Stuck in Terminating

Remove finalizers manually:

```bash
kubectl patch microfrontend <name> -n <namespace> \
  -p '{"metadata":{"finalizers":null}}' --type=merge
```

## Uninstalling

To remove Polyfea:

```bash
# 1. Delete all custom resources first
kubectl delete microfrontendclass,microfrontend,webcomponent --all-namespaces --all

# 2. Uninstall the Helm releases
helm uninstall polyfea-samples -n polyfea
helm uninstall polyfea-controller -n polyfea-system

# 3. Delete the namespaces (optional)
kubectl delete namespace polyfea polyfea-system
```

## Additional Resources

- [Helm Chart Repository](https://github.com/polyfea/charts)
- [Controller Documentation](https://github.com/polyfea/polyfea-controller)
- [Browser Library (@polyfea/core)](https://www.npmjs.com/package/@polyfea/core)
- [Example Applications](https://github.com/polyfea/examples)
