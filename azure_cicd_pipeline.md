# Azure CI/CD Pipeline with GitHub, Jenkins, and Kubernetes

This project demonstrates how to set up a CI/CD pipeline using Azure, GitHub, Jenkins, and Kubernetes for deploying a Node.js application.

## Prerequisites
- GitHub account
- Jenkins installed with required plugins
- Docker installed and configured
- Kubernetes cluster (Azure Kubernetes Service preferred)
- Azure CLI installed

## Setup Instructions

### 1. Upload Your Node.js Application to GitHub

```sh
cd /path/to/your-project
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/your-username/your-repo.git
git branch -M main
git push -u origin main
```

### 2. Set Up Jenkins for CI/CD

#### Install Required Plugins:
- Docker Plugin
- Kubernetes Plugin
- Pipeline Plugin
- Git Plugin

#### Configure GitHub Webhook:
1. Navigate to **Settings > Webhooks** in your GitHub repository.
2. Add a webhook with the URL: `http://your-jenkins-server/github-webhook/`
3. Set **Content type** to `application/json`.
4. Enable **Push events** and click **Add webhook**.

### 3. Create a Jenkinsfile for Automation

```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME = "fffeeeeeee/nodejs-app:latest"
        KUBE_CONFIG = credentials('k8s-credentials') // Add K8s credentials in Jenkins
    }
    stages {
        stage('Clone Repo') {
            steps {
                git url:'https://github.com/hilalahmad0101/nodejs-k8s-azure.git',branch:'master'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t fffeeeeeee/nodejs-app:latest .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                    sh 'docker push $IMAGE_NAME'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'k8s-credentials', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s/deployment.yml'
                    sh 'kubectl apply -f k8s/service.yml'
                    sh 'kubectl rollout restart deployment nodejs-app'
                    sh 'kubectl rollout status deployment nodejs-app'
                    sh 'kubectl get pods'
                }
            }
        }
    }
    post {
        success {
            emailext(
                subject: "✅ Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "The Jenkins build has completed successfully.\n\nView details: ${env.BUILD_URL}",
                recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider']],
                to: 'hilal@archiwiz.com'
            )
        }
        failure {
            emailext(
                subject: "❌ Jenkins Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "The Jenkins build has FAILED.\n\nCheck logs: ${env.BUILD_URL}",
                recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'DevelopersRecipientProvider']],
                to: 'hilal@archiwiz.com'
            )
        }
    }
}
```

### 4. Dockerize Your Application

Create a `Dockerfile` in the root of your project:

```dockerfile
FROM node:16
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
EXPOSE 3000
```

### 5. Set Up Kubernetes Configuration

Create a `k8s/` directory and add the following files:

#### Deployment Configuration (`deployment.yml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
        - name: node-app
          image: your-dockerhub-username/your-app:latest
          ports:
            - containerPort: 3000
```

#### Service Configuration (`service.yml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-app-service
spec:
  selector:
    app: node-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

### 6. Connect to Azure Using the Terminal

#### Login to Azure:
```sh
az login
```

#### Set the Subscription (If Multiple Subscriptions Exist):
```sh
az account set --subscription "your-subscription-id"
```

#### Connect to the AKS Cluster:
```sh
az aks get-credentials --resource-group your-resource-group --name your-aks-cluster
```

#### Deploy the Application to Kubernetes:
```sh
kubectl apply -f k8s/
```

#### Verify Deployment:
```sh
kubectl get pods
kubectl get services
```

## Conclusion
By following these steps, you can successfully automate the CI/CD process for deploying your Node.js application using Azure, GitHub, Jenkins, and Kubernetes.
