# CI/CD Pipelines with Jenkins, Maven, Docker, Nexus, SonarQube, Tomcat, and Kubernetes (EKS)
# INFRASTRUCTURE SETUP – AWS EC2 & EKS

## 🔹 Operating System
- **Amazon Linux AMI** used for Jenkins and EKS host instances.


## 🔹 EC2 Instances
| Service   | Instance Name | Purpose                                | Default Port |
|-----------|---------------|----------------------------------------|--------------|
| Jenkins   | `jenkins`     | CI/CD pipeline execution               | 8080         |
| EKS       | `eks-cluster` | Cluster creation & management (kubectl, eksctl, AWS CLI) | 22 (SSH)     |
| Docker Hub| Cloud Service | Artifact repository for container 
## 🔹 Security Group – Inbound Rules
- **22 (SSH)** → Restricted to your IP only  
- **8080 (Jenkins)** → Open for Jenkins UI access
# JENKINS SETUP

## 🔹 Installation
- Jenkins installed on a dedicated **EC2 instance** (`jenkins`) running Amazon Linux AMI.  
- Service started and verified.  
- Jenkins UI accessible at:  http://13.126.197.139:8080/
![image alt](https://github.com/Jenifa68jeni/jenkins-ci-cd-eks/blob/2fec336171e63b52b1cfd3e76f809bbf8850e98d/Screenshot%202026-03-31%20170243.png)
 
## 🔹 Global Tool Configuration
- Navigate to: **Manage Jenkins → Global Tool Configuration**  
- Configured Maven installation:  
- Name: `maven`  
- Version: `3.8.4`  
- Docker installed on Jenkins server to enable container image creation and pushing to Docker Hub.  
- Jenkins user added to Docker group for pipeline execution.
# JENKINS CREDENTIALS SETUP

## 🔹 Purpose
Credentials in Jenkins are used to securely store sensitive information such as Docker Hub login, SSH keys, and cloud access tokens. These are then referenced in pipeline stages without exposing them in code.

---

## 🔹 Stored Credentials
- **Docker Hub Credentials**
  - ID: `dockerhub-credentials`
  - Used by Jenkins pipeline to push Docker images into Docker Hub.
  
- **SSH Agent Credentials**
  - Used for secure communication with EC2/EKS host VM if required.
# EKS HOST VM SETUP

## 🔹 Purpose
The EKS Host VM was launched and configured to manage the Kubernetes cluster.  
Installed tools:
- **AWS CLI** → Connect Jenkins with AWS EKS  
- **kubectl** → Interact with the Kubernetes cluster  
- **eksctl** → Create and manage EKS clusters  
  
- **IAM Role (Attached to Jenkins EC2 Instance)**
  - Permissions included:
    - AmazonEC2ContainerRegistryReadOnly  
    - AmazonEC2FullAccess  
    - AmazonEKSClusterPolicy  
    - AmazonEKSWorkerNodePolicy
    - 
# CLUSTER CREATION
eksctl create cluster \
  --name jenifa-cluster \
  --region ap-south-1 \
  --node-type c7i-flex.large \
  --zones ap-south-1a,ap-south-1b
![image alt](https://github.com/Jenifa68jeni/jenkins-ci-cd-eks/blob/27683bebcc8028bfab9803701b117d3458d8c94d/Screenshot%202026-03-31%20171240.png)

##🔹 Verification
![image alt](https://github.com/Jenifa68jeni/jenkins-ci-cd-eks/blob/4e81d36f82b105a2b8eba1bdb0e573a7a7fdf939/Screenshot%202026-03-31%20171443.png)
# JENKINS PIPELINE INTEGRATION WITH KUBERNETES (EKS)

## 🔹 IAM Role Setup (Jenkins VM)
Attached IAM role to the Jenkins EC2 instance with permissions:
- AmazonEC2ContainerRegistryReadOnly  
- AmazonEC2FullAccess  
- AmazonEKSClusterPolicy  
- AmazonEKSWorkerNodePolicy  

This allows Jenkins to communicate securely with EC2 and EKS services for deployment operations.
## Verification:
![image alt](https://github.com/Jenifa68jeni/jenkins-ci-cd-eks/blob/f73fd112fe683e55d7fb28385de831fff02debc9/Screenshot%202026-03-31%20172007.png)
# KUBERNETES DEPLOYMENT SETUP (JENKINS CLUSTER CONNECTION)

## 🔹 Required Tools on Jenkins Server
- **AWS CLI** → Connect Jenkins with AWS EKS  
- **kubectl** → Interact with the Kubernetes cluster  
- **eksctl** → Manage EKS cluster creation and scaling  



 🔹 Kubeconfig Setup
- Copy the `.kube/config` file generated during cluster creation into Jenkins server.  
- Place it inside `/var/lib/jenkins/qube/config`.  
- Verified cluster connectivity with:  
  kubectl get nodes
  
🔹 Deployment Manifest (deployment.yaml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: maven-web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: maven-web-app
  template:
    metadata:
      labels:
        app: maven-web-app
    spec:
      containers:
      - name: maven-web-app
        image: jenifajeni/maven-web-app:latest
        ports:
        - containerPort: 8080
        
🔹 Service Manifest (service.yaml)
  apiVersion: v1
kind: Service
metadata:
  name: maven-web-app
spec:
  type: LoadBalancer
  selector:
    app: maven-web-app
  ports:
  - port: 80
    targetPort: 8080
    
🔹 Jenkins Deployment Stage
stage('Kubernetes Deployment') {
    steps {
        sh '''
            kubectl apply -f deployment.yaml
            kubectl apply -f service.yaml
            kubectl get pods
            kubectl get svc
        '''
    }
 }
 
🔹 Verification

![image alt](https://github.com/Jenifa68jeni/jenkins-ci-cd-eks/blob/b2344e86c1cf43c442de1eb512863931f21c4c62/Screenshot%202026-03-31%20172611.png)

![image alt](https://github.com/Jenifa68jeni/jenkins-ci-cd-eks/blob/a1c2ff95968d0cff9891b583e4535519bcf9945f/Screenshot%202026-03-31%20172749.png)

![image alt](https://github.com/Jenifa68jeni/jenkins-ci-cd-eks/blob/f4d6947b255c71a2f92c85b6a70e2f80a6b4ee0a/Screenshot%202026-03-31%20172907.png)




 
