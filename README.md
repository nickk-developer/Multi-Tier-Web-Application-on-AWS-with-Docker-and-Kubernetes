# Multi-Tier Web Application on AWS with Docker and Kubernetes

This project demonstrates how to deploy a multi-tier web application on AWS using Docker and Kubernetes. The application consists of a frontend (React) and a backend (Node.js or Flask), with the backend connected to an AWS RDS database.

---

## Prerequisites

Before starting the project, ensure you have the following:

- An AWS account
- Docker installed ([Download Docker](https://www.docker.com/get-started))
- **kubectl** installed ([Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/))
- AWS CLI installed ([Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html))

---

## Project Structure

```bash
multi-tier-web-app-aws/
├── backend-api/
│   ├── Dockerfile
│   └── app.js
├── frontend-app/
│   ├── Dockerfile
│   └── src/
├── frontend-deployment.yaml
├── backend-deployment.yaml
├── README.md
├── .gitignore
```

---

## Steps to Build and Deploy the Application

### Step 1: Set Up Docker Environment

**Explanation:**  
Docker allows us to containerize the application, ensuring that it runs consistently across various environments.

**Actions:**

- **Install Docker:** Download Docker Desktop from Docker's official website and follow the installation instructions.
- **Verify Docker Installation:** Open your terminal and run the following command to check if Docker is installed:
  ```bash
  docker --version
  ```

---

### Step 2: Create the Application (Frontend and Backend)

**Frontend:**

- **Create a Frontend App:**
  ```bash
  mkdir frontend-app
  cd frontend-app
  npx create-react-app .
  ```

- **Run the App:**
  ```bash
  npm start
  ```

- **Verify in Browser:** Open `http://localhost:3000`.

**Backend:**

- **Create a Backend API Directory:**
  ```bash
  mkdir ../backend-api
  cd ../backend-api
  ```

- **For Node.js Backend (Using Express):**

  - **Install Express and Create a Simple API:**
    ```bash
    npm init -y
    npm install express
    ```

  - **Create app.js:**
    ```javascript
    const express = require('express');
    const app = express();
    const port = 5000;

    app.get('/', (req, res) => {
      res.send('Hello from the Backend API!');
    });

    app.listen(port, () => {
      console.log(`Backend API running on port ${port}`);
    });
    ```

  - **Run the Backend:**
    ```bash
    node app.js
    ```

---

### Step 3: Dockerize the Frontend and Backend

**Frontend Dockerfile:**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

**Backend Dockerfile (Node.js):**
```dockerfile
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 5000
CMD ["node", "app.js"]
```

**Build Docker Images:**
- Frontend:
  ```bash
  docker build -t frontend-app .
  ```

- Backend:
  ```bash
  docker build -t backend-api .
  ```

---

### Step 4: Push Docker Images to Amazon ECR (Elastic Container Registry)

**Actions:**

- **Create an ECR Repository:** Create repositories for `frontend-app` and `backend-api` in the AWS Console.
- **Authenticate Docker to ECR:**
  ```bash
  aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
  ```

- **Tag and Push Docker Images:**
  ```bash
  docker tag frontend-app:latest <account-id>.dkr.ecr.<region>.amazonaws.com/frontend-app:latest
  docker push <account-id>.dkr.ecr.<region>.amazonaws.com/frontend-app:latest

  docker tag backend-api:latest <account-id>.dkr.ecr.<region>.amazonaws.com/backend-api:latest
  docker push <account-id>.dkr.ecr.<region>.amazonaws.com/backend-api:latest
  ```

---

### Step 5: Set Up Amazon EKS Cluster

- **Create an EKS Cluster**: In AWS EKS Console, create a new EKS cluster and configure the VPC and subnets.
- **Install kubectl:** Use kubectl to manage your Kubernetes cluster:
  ```bash
  aws eks --region <region> update-kubeconfig --name <cluster-name>
  ```

---

### Step 6: Deploy Frontend and Backend Services to Kubernetes

**Frontend Deployment YAML:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: <account-id>.dkr.ecr.<region>.amazonaws.com/frontend-app:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 3000
```

**Backend Deployment YAML:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: <account-id>.dkr.ecr.<region>.amazonaws.com/backend-api:latest
        ports:
        - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 5000
      targetPort: 5000
```

**Apply Deployments:**
```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-deployment.yaml
```

---

### Step 7: Connect Backend to RDS

1. **Create an RDS Instance** in AWS RDS Console.
2. **Modify Backend Code** to connect to RDS using environment variables in `backend-deployment.yaml`.

---

### Step 8: Set Up Auto-Scaling and Monitoring

**Enable Horizontal Pod Autoscaler:**
```bash
kubectl autoscale deployment frontend-deployment --cpu-percent=50 --min=1 --max=5
kubectl autoscale deployment backend-deployment --cpu-percent=50 --min=1 --max=5
```

**Enable CloudWatch Logs for EKS.**

---

### Step 9: Test the Application

- Access the frontend using the LoadBalancer IP.
- Use Postman or `curl` to interact with the backend API.
- Test the integration with the RDS database.

---
