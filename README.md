# Project_ST2SCL_Cloud

## Service 1 : backend

### Install Docker and Test service1 using Docker

Start the container: docker run -p 4000:8080 -t gaoweicao/service1:1

8080 is the port of the web service, while 4000 is the port for accessing the container. Test the web service using a web browser: http://localhost:4000 It displays hello.

### Install kubernetes-minikube and start it with Docker

Minikube is a tool that lets you run Kubernetes locally. 
minikube runs a single-node Kubernetes cluster on your personal computer (including Windows, macOS and Linux PCs) so that you can try out Kubernetes, or for daily development work.

https://minikube.sigs.k8s.io/docs/start/

minikube start --driver=docker

Minikube provides a dashboard (web portal). Access the dashboard using the following command:
 
minikube dashboard

### Create a kubernetes deployment from Docker image : gaoweicao/service1:1

kubectl get nodes

kubectl create deployment service1 --image=gaoweicao/service1:1

### Expose HTTP and HTTPS route using NodePort

kubectl expose deployment service1 --type=NodePort --port=8080

Retrieve the service address: minikube service service1 --url

This format of this address is NodeIP:NodePort.

Test this address inside your browser. It should display hello again.

## Service 2 : frontend

### Create a kubernetes deployment from Docker image : gaoweicao/service2:1

kubectl apply -f front-back-app.yml
