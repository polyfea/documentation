# Containerizing Your Web Component

In this tutorial, you'll package your web components into a lightweight Docker image using nginx, then push it to a container registry. This makes your microfrontend ready for Kubernetes deployment.

## What You'll Build

- A production-ready Docker image with nginx serving your web components
- Optimized image size by including only runtime dependencies
- A containerized microfrontend ready for deployment

## Prerequisites

- Completed the [Building a Web Component](webcomponents.md) tutorial
- Docker installed and running on your machine
- A Docker Hub account (or access to another container registry)
- Built web components from the previous tutorial (`npm run build` completed)

## Step 1: Create the Dockerfile

In your project directory (from the previous tutorial), create a `Dockerfile`:

```Dockerfile
# Use nginx alpine image for lightweight serving
FROM nginx:alpine

COPY dist/webcomponents.bundled.js /usr/share/nginx/html/webcomponents.bundled.js

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Understanding the Dockerfile

- **nginx:alpine**: Uses the smallest nginx base image (~23MB), perfect for serving static files
- **JavaScript Files**: Copies your built web components
- **Port 80**: Standard HTTP port for web serving
- **Daemon Off**: Keeps nginx running in the foreground for Docker

## Step 2: Create a .dockerignore File

Optimize your build by excluding unnecessary files. Create `.dockerignore` in the project root:

```plaintext
node_modules/*
!node_modules/@webcomponents
!node_modules/lit
src/
test/
docs/
docs-src/
.git
.gitignore
*.md
tsconfig.json
rollup.config.js
web-dev-server.config.js
web-test-runner.config.js
package-lock.json
```

### Why .dockerignore?

- **Faster Builds**: Reduces context size sent to Docker daemon
- **Smaller Images**: Excludes development files from the final image
- **Security**: Prevents accidentally copying sensitive files
- **Selective Inclusion**: The `!node_modules/...` syntax includes only specific runtime dependencies

## Step 3: Build the Docker Image

Build your image with a descriptive tag:

```bash
docker build -t <your-username>/my-web-components:latest .
```

**Replace `<your-username>`** with your Docker Hub username (e.g., `johnsmith/my-web-components:latest`).

### Build Output

You should see output similar to:

```
[+] Building 2.3s (10/10) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 234B
 => [internal] load .dockerignore
 => [1/3] FROM docker.io/library/nginx:alpine
 => [2/3] COPY *.js /usr/share/nginx/html/
 => [3/3] COPY node_modules/@webcomponents ...
 => exporting to image
 => => naming to docker.io/<your-username>/my-web-components:latest
```

Verify the image was created:

```bash
docker images | grep my-web-components
```

## Step 5: Push to Container Registry

First, log in to Docker Hub:

```bash
docker login
```

Enter your Docker Hub username and password when prompted.

Push your image:

```bash
docker push <your-username>/my-web-components:latest
```

### Push Progress

You'll see upload progress for each layer:

```
The push refers to repository [docker.io/<your-username>/my-web-components]
a1b2c3d4e5f6: Pushed
f6e5d4c3b2a1: Pushed
latest: digest: sha256:abc123... size: 1234
```

## What's Next?

Your Docker image is now ready and available in a container registry! You've successfully:

- Created a lightweight nginx-based container
- Optimized the image with .dockerignore
- Built and tested the Docker image
- Pushed it to a container registry

In the [next tutorial](k8s-manifests.md), you'll learn how to:

- Write Kubernetes Deployment and Service manifests
- Create Polyfea Custom Resources (MicroFrontendClass, MicroFrontend, WebComponent)
- Deploy your microfrontend to a local Kubernetes cluster
- Test the complete Polyfea integration