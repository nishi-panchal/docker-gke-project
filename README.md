# Next.js App Deployment with Docker, GKE and ArgoCD

A step-by-step guide to deploying a Next.js app using Docker, Google Kubernetes Engine (GKE), and ArgoCD for continuous deployment.

## What We Built

- Next.js app with TypeScript
- Containerized the app using Docker
- Deployed to Google Cloud using GKE
- Set up automatic deployments with ArgoCD

## Prerequisites

- Node.js installed
- Docker Desktop
- Google Cloud account
- Basic knowledge of TypeScript and React

## Step 1: Create Next.js App

```bash
npx create-next-app@latest my-app
cd my-app
```

## Step 2: Containerize with Docker

Create a `Dockerfile` in your project root:

```dockerfile
# Build stage
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm install

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS runner
WORKDIR /app
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"
CMD ["node", "server.js"]
```

Build and test locally:
```bash
docker build -t nextjs-app .
docker run -p 3000:3000 nextjs-app
```

## Step 3: Deploy to GKE

Create deployment files:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextjs-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nextjs
  template:
    metadata:
      labels:
        app: nextjs
    spec:
      containers:
      - name: nextjs
        image: gcr.io/[PROJECT_ID]/nextjs-app:v1
        ports:
        - containerPort: 3000
```

## Step 4: Set Up ArgoCD

1. Install ArgoCD on your cluster
2. Access ArgoCD UI
3. Connect your Git repository
4. Configure automatic sync

## Common Issues We Faced

1. Docker Desktop not running
   - Solution: Start Docker Desktop before building

2. PowerShell commands not working
   - Solution: Use correct string formatting for PowerShell

3. ArgoCD UI access issues
   - Solution: Use correct port forwarding commands

## Useful Commands

```bash
# Build Docker image
docker build -t nextjs-app .

# Deploy to Kubernetes
kubectl apply -f deployment.yaml

# Check deployment status
kubectl get deployments
kubectl get pods
```

## What We Learned

1. Docker basics:
   - How to build images
   - Multi-stage builds
   - Container management

2. Kubernetes concepts:
   - Deployments
   - Services
   - Pods

3. CI/CD practices:
   - Automated deployments
   - Version control
   - Container registry usage

## Next Steps

- Add monitoring
- Set up staging environment
- Implement automated testing
- Add SSL/TLS

## Project Structure
```
my-app/
├── app/                # Next.js files
├── kubernetes/        # Kubernetes configs
├── Dockerfile        # Docker config
└── README.md        # Documentation
```

