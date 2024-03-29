# Project_ST2SCL_Cloud

A web app composed of two web services:

- a frontend
- a backend

## Service 1 : backend

The backend is coded in Java (Spring boot), but it doesn't matter. The Web service simply return "Hello from Service 1 !" when it receives an HTTP Get on "/": https://github.com/gcao2405/Project_ST2SCL_Cloud/blob/main/Service1/src/main/java/com/example/Service1/MyWebService.java

A Docker image is already available is the docker hub at: https://hub.docker.com/u/gaoweicao, and this project uses it.

### Install Docker and Test service1 using Docker

Start the container: 

```
docker run -p 4000:8080 -t gaoweicao/service1:1
```

8080 is the port of the web service, while 4000 is the port for accessing the container. Test the web service using a web browser: http://localhost:4000 It displays hello.

### Install kubernetes-minikube and start it with Docker

Minikube is a tool that lets you run Kubernetes locally. 
minikube runs a single-node Kubernetes cluster on your personal computer (including Windows, macOS and Linux PCs) so that you can try out Kubernetes, or for daily development work.

https://minikube.sigs.k8s.io/docs/start/

```
minikube start --driver=docker
```

Minikube provides a dashboard (web portal). Access the dashboard using the following command:

```
minikube dashboard
```

### Create a kubernetes deployment from Docker image : gaoweicao/service1:1

```
kubectl get nodes

kubectl create deployment service1 --image=gaoweicao/service1:1

```

### Expose HTTP and HTTPS route using NodePort

```
kubectl expose deployment service1 --type=NodePort --port=8080
```

Retrieve the service address: 

```
minikube service service1 --url
```

This format of this address is NodeIP:NodePort.

Test this address inside your browser. It should display hello.

## Service 2 : frontend

The frontend (also coded in Java Spring boot) waits for an HTTP get on "/". Then it requests the backend service:

```
restTemplate restTemplate = new RestTemplate();
String s = restTemplate.getForObject(backEndURL, String.class);
```

and concatenates and returns the result "Hello from Service 1 !" with "Hello": 

```
return "hello (from the front end)" + " " + s + " (from the back end)";
```

https://github.com/gcao2405/Project_ST2SCL_Cloud/blob/main/Service2/src/main/java/com/example/Service2/MyWebService.java

A Docker image is already available is the docker hub at: https://hub.docker.com/u/gaoweicao, and this project uses it.

### Create a kubernetes deployment from Docker image : gaoweicao/service2:1

```
kubectl create deployment service2 --image=gaoweicao/service2:1
```

### Kubernestes configuration file

A single yaml file is enough to configure the cluster: https://github.com/gcao2405/Project_ST2SCL_Cloud/blob/main/front-back-app.yml

Yaml files can be used instead of using the command "kubectl create deployment" and "kubectl expose deployment"

```
kubectl apply -f front-back-app.yml
```

This yaml file contains:

#### The frontend deployment and the frontend service in front of the deployment

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that’s outside of your cluster.

Kubernetes ServiceTypes allow you to specify what kind of Service you want. The default is ClusterIP.

The type for the frontend is set to NodePort. His behaviors are to expose the Service on each Node’s IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You’ll be able to contact the NodePort Service, from outside the cluster, by requesting NodeIP:NodePort.

#### The backend deployment and the backend service in front of the deployment

The type set to ClusterIP. A Kubernetes Service is an abstraction which defines a logical set of Pods running somewhere in the cluster, that all provide the same functionality. When created, each Service is assigned a unique IP address (also called clusterIP). This address is tied to the lifespan of the Service, and will not change while the Service is alive.

### Service discovery: how can the frontend reach the backend?

Kubernetes supports 2 primary modes of finding a Service - environment variables and DNS (Domain Name Server). The former works out of the box while the latter requires the CoreDNS cluster addon.

The backend address is injected inside the frontend app (https://github.com/gcao2405/Project_ST2SCL_Cloud/blob/main/Service2/src/main/java/com/example/Service2/MyWebService.java) with:
```
@Value("${backEndURL}")
String backEndURL;
```

### Test the app from a Web browser

Since the kind of the front-end service is NodePort you can get its IP address with:

```
minikube service service2 --url
```

Then test the app with a Web browser.
It should display 'hello (from the front end) Hello from Service 1 ! (from the back end)'

## Routing rule to a service using Ingress

You can use Ingress to expose your Service. Ingress is not a Service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address. Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting.

### Set up Ingress on Minikube with the NGINX Ingress Controller

Enable the NGINX Ingress controller:
```
minikube addons enable ingress
```

On Windows : edit the c:\windows\system32\drivers\etc\hosts file, add
```
127.0.0.1 front-end.localhost
```

Enable a tunnel for Minikube:
```
minikube addons enable ingress-dns
```
```
minikube tunnel
```

Then apply the yaml file again : 
```
kubectl apply -f front-back-app.yml
```

Check now in your Web browser for the front-end service :
```
http://front-end.localhost/service2
```
For the back-end service :
```
http://front-end.localhost/service1
```
