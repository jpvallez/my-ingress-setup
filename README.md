# I'm Learning about Ingress!

I have a minikube cluster setup with an Ingress config (yaml) and an Ingress controller (ingress-nginx)


# My Setup

1. Install kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl/

2. Install minikube
https://kubernetes.io/docs/tasks/tools/install-minikube/


3. Start minikube
```
minikube start --driver=docker
```
>:question: ```--driver=docker``` uses my system's docker instead of running minikube VM with a Hypervisor.

Output:
```
minikube v1.13.0 on Linuxmint 20  
Using the docker driver based on user configuration  
Starting control plane node minikube in cluster minikube  
Pulling base image ...  
Downloading Kubernetes v1.19.0 preload ...  
> preloaded-images-k8s-v6-v1.19.0-docker-overlay2-amd64.tar.lz4: 486.28 MiB  
Creating docker container (CPUs=2, Memory=3900MB) ...  
Preparing Kubernetes v1.19.0 on Docker 19.03.8 ...  
Verifying Kubernetes components...  
Enabled addons: default-storageclass, storage-provisioner  
Done! kubectl is now configured to use "minikube" by default  
```
:metal:

<br/>

3. Enable Ingress plugin on minikube. This adds an ingress-nginx pod to the system.
```
minikube addons enable ingress
```
<br/>

4. Deploy and expose 2 simple web apps (any image you like)

```
Kubectl create deployment web1 --image=<image-registry-address>
```
```
kubectl expose deployment web1 --type=NodePort --port=8080
```
>(and once more, replacing web1 with web2)

<br/>

5. Create Ingress Config yaml.

The below yaml specifies an Ingress config to redirect traffic accessing ```hello-world.info/``` to our web1 service, and ```hello-world.info/v2``` to our web2 service

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - host: hello-world.info
    http:
      paths:
      - path: /
        backend:
          serviceName: web1
          servicePort: 8080
      - path: /v2
        backend:
          serviceName: web2
          servicePort: 8080
```
> :question: ```hello-world.info``` can be added to your /etc/hosts file to point at the IP of your ingress controller outputted by: ```> kubectl get ingress```

<br/>

6. Apply the Ingress Config so the Ingress Controller can enforce the rule.
```
kubectl apply -f example-ingress.yaml
```
<br/>

7. Ingress Controller now directs requests for "/" to web1 service/cluster and requests for "/v2" to web2 service/cluster.
    * Requests hit Ingress Controller IP
    * Ingress Controller responds with services endpoints.