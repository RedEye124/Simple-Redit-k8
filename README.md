Simple Reddit Application Deployment on Kubernetes
This guide outlines the steps to deploy a simple Reddit application using Kubernetes. Follow these instructions to set up your environment and deploy the application.

Prerequisites
EC2 Instances

CI Server: t2.micro instance
Deployment Server: t2.medium instance
Software

Docker
Minikube
kubectl
Steps
_____________________________________________________________________________________________________________________________________________________________________________________
1. Create EC2 Instances
Create two EC2 instances with the following specifications:

CI Server:
Instance Type: t2.micro
AMI: Ubuntu
Deployment Server:
Instance Type: t2.medium
AMI: Ubuntu
2. Clone the Repository on CI Server
SSH into the CI server and clone the GitHub repository for the Reddit application source code:   git clone https://github.com/LondheShubham153/reddit-clone-k8s-ingress.git
________________________________________________________________________________________________________________________________________________________________________________
3. Install Docker on CI Server
Install Docker on the CI server using the following commands:
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
___________________________________________________________________________________________________________________________________________________________________________________
4. Create a Dockerfile
Create a Dockerfile in the cloned repository directory with the following content:
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone

RUN npm install

EXPOSE 3000

CMD ["npm", "run", "dev"]

__________________________________________________________________________________________________________________________________________________________________________________

5. Build the Docker Image
Build the Docker image from the Dockerfile:

docker build -t <DockerHub_Username>/<ImageName> .
___________________________________________________________________________________________________________________________________________________________________________________

6. Push the Docker Image to DockerHub
First, log in to your DockerHub account:

docker login

Then push the image to DockerHub:

docker push <DockerHub_Username>/<ImageName>

___________________________________________________________________________________________________________________________________________________________________________________

7. Install Docker and Kubernetes on Deployment Server
SSH into the deployment server and install Docker and Minikube:
sudo apt-get update
sudo apt-get install docker.io -y
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start


Install kubectl:
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
__________________________________________________________________________________________________________________________________________________________________________


8. Create Deployment Configuration
Create a file named deployment.yml with the following content:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: <DockerHub_Username>/<ImageName>
        ports:
        - containerPort: 3000
Apply the deployment file:
kubectl apply -f deployment.yml
kubectl get deployments
____________________________________________________________________________________________________________________________________________________________________________

9. Create Service Configuration
Create a file named service.yml with the following content:
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone




Apply the service file:
kubectl apply -f service.yml
minikube service reddit-clone-service --url
curl http://<Minikube_IP>:31000
_____________________________________________________________________________________________________________________________________________________________________________

10. Expose the Application
Method 1: NodePort
Expose the deployment using NodePort:

kubectl expose deployment reddit-clone-deployment --type=NodePort

Test your deployment:

curl -L http://<Minikube_IP>:31000

Method 2: Ingress
Enable the ingress add-on in Minikube:
minikube addons enable ingress
Create a file named ingress.yml with the following content:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000

Apply the ingress file:


kubectl apply -f ingress.yml
curl -L domain.com/test
_______________________________________________________________________________________________________________________________________________________________________________________

11. Security Group Configuration
To access the application externally, ensure that the security group associated with your deployment server allows inbound traffic on port 31000.

This setup will allow you to deploy and access your Reddit application using Kubernetes.



______________________________________________________________________________________________________________________________________________________________________________

