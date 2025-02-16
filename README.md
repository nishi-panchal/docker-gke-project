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
![Screenshot 2025-01-10 220901](https://github.com/user-attachments/assets/e7bb4be0-f4c0-4e8f-b6fe-fc9e174444d0)
![Screenshot 2025-01-10 221809](https://github.com/user-attachments/assets/fba40f9f-80b6-4237-82fc-72d8b7d4a241)

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
![Screenshot 2025-01-11 000014](https://github.com/user-attachments/assets/211cf45a-a19c-4a23-8028-ae65468a9a06)
![Screenshot 2025-01-11 000354](https://github.com/user-attachments/assets/0a7bc54f-e784-45bf-b432-d93e28202e44)
![Screenshot 2025-01-11 000412](https://github.com/user-attachments/assets/ae109ad0-7f0a-4e8a-a866-d6f12518829f)

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
![Screenshot 2025-01-11 003800](https://github.com/user-attachments/assets/64d8fb17-feb0-4824-937c-86fd68faaef0)
![Screenshot 2025-01-11 004944](https://github.com/user-attachments/assets/fabc89e2-df19-4567-bfe7-eb1f35e55371)

## Step 4: Set Up ArgoCD

![Screenshot 2025-01-11 014504](https://github.com/user-attachments/assets/3a2748cc-c7e1-4907-97f0-c65c94c182fd)

1. Install ArgoCD on your cluster
2. Access ArgoCD UI
3. Connect your Git repository
4. Configure automatic sync

![Screenshot 2025-01-11 021014](https://github.com/user-attachments/assets/7d8a92f2-4410-4140-96a4-86d754caa76c)

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

