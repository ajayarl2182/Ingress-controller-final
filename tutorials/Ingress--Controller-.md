---
title: Sample Tutorial
description: This is a sample tutorial to guide user on creating his own tutorial in md.User should follow exact syntax given below to render in user guide
---

### Introduction
**What is Ingress?**
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
![Ingress.png](_images/Ingress.png)
Here is a simple example where an Ingress sends all its traffic to one Service:

An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.
An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type Service.Type=NodePort or Service.Type=LoadBalancer.


**Prerequisites:**

You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.
You may need to deploy an Ingress controller such as ingress-nginx. You can choose from a number of Ingress controllers.
Ideally, all Ingress controllers should fit the reference specification. In reality, the various Ingress controllers operate slightly differently.
![Prerequisites.png](_images/Prerequisites.png)

**The Ingress resource:**

I**ngress resource example:**

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80


As with all other Kubernetes resources, an Ingress needs apiVersion, kind, and metadata fields. The name of an Ingress object must be a valid DNS subdomain name. For general information about working with config files, see deploying applications, configuring containers, managing resources. Ingress frequently uses annotations to configure some options depending on the Ingress controller, an example of which is the rewrite-target annotation. Different Ingress controller support different annotations. Review the documentation for your choice of Ingress controller to learn which annotations are supported.
The Ingress spec has all the information needed to configure a load balancer or proxy server. Most importantly, it contains a list of rules matched against all incoming requests. Ingress resource only supports rules for directing HTTP(S) traffic.

**Ingress rules:**
 
Each HTTP rule contains the following information:


* An optional host. In this example, no host is specified, so the rule applies to all inbound HTTP traffic through the IP address specified. If a host is provided (for example, foo.bar.com), the rules apply to that host.
* A list of paths (for example, /testpath), each of which has an associated backend defined with a service.name and a service.port.name or service.port.number. Both the host and path must match the content of an incoming request before the load balancer directs traffic to the referenced Service.
* A backend is a combination of Service and port names as described in the Service doc or a custom resource backend by way of a CRD. HTTP (and HTTPS) requests to the Ingress that matches the host and path of the rule are sent to the listed backend.
A defaultBackend is often configured in an Ingress controller to service any requests that do not match a path in the spec.

**Ingress class:**

Ingresses can be implemented by different controllers, often with different configuration. Each Ingress should specify a class, a reference to an IngressClass resource that contains additional configuration including the name of the controller that should implement the class.

apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb
spec:
  controller: example.com/ingress-controller
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb

IngressClass resources contain an optional parameters field. This can be used to reference additional configuration for this class.


**Default IngressClass:
**
You can mark a particular IngressClass as default for your cluster. Setting the ingressclass.kubernetes.io/is-default-class annotation to true on an IngressClass resource will ensure that new Ingresses without an ingressClassName field specified will be assigned this default IngressClass.


Note: If you have more than one IngressClass marked as the default for your cluster, the admission controller prevents creating new Ingress objects that don't have an ingressClassName specified. You can resolve this by ensuring that at most 1 IngressClass is marked as default in your cluster.

**Updating an Ingress**:

To update an existing Ingress to add a new Host, you can update it by editing the resource:



### How to create and manage Ingress Controller 

Deploy a sample Hello App using the below commands 

kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:1.0

Kubectl get pod

Pods are deployed in default namespace

Once the hello-app deployed expose the app to connect from cluster from port 8080 on default namespace

kubectl expose deployment hello-app  --port=8080

Kubectl get svc

Service deployed on default namespace

Deploy ingress controller using node port service to connect from outside cluster

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/baremetal/deploy.yaml

Ingress controller will deployed on ingress-nginx namespace


Kubectl get pods -n ingress-nginx

Kubectl get svc -n ingress-nginx


**Sample hello-app ingress example.**

cat << EOF | kubectl apply -f -   
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: hello-app
spec:
  rules:
  - http:
      paths:
      - path: /hello
        backend:
          serviceName: hello-app
          servicePort: 8080
EOF

Hello app ingress is creating on default namespace.

Kubectl get ing

It will display the ingress address to connect from browser

We are using node port service so we can able to access the site using node port of the ingress controller which we can get it from 

Kubectl get svc -n ingress-nginx  eg: 80:31688/TCP,443:3118

http://<EXTERNAL_IP>:31688/hello

URL :  http://##DNS.IP##:30455

**Types of Ingress**
1- Single Service Ingress
2-Simple fanout
3-Name based virtual hosting
4-Loadbalancing

**Single Service Ingress:**
There are existing Kubernetes concepts that allow you to expose a single service (see alternatives), however you can do so through an Ingress as well, by specifying a default backend with no rules.

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
spec:
  backend:
    serviceName: testsvc
    servicePort: 80

**Simple fanout:**
As described previously, pods within kubernetes have ips only visible on the cluster network, so we need something at the edge accepting ingress traffic and proxying it to the right endpoints. This component is usually a highly available loadbalancer/s. An Ingress allows you to keep the number of loadbalancers down to a minimum, for example, a setup like:
![Simple fanout.png](_images/Simple fanout.png)

would require an Ingress such as:

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foo
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bar
        backend:
          serviceName: s2
          servicePort: 80

When you create the Ingress with kubectl create -f
![Fanout.png](_images/Fanout.png)

Name based virtual hosting:
Name-based virtual hosts use multiple host names for the same IP address.
![Name based virtual hosting-1.png](_images/Name based virtual hosting-1.png)
The following Ingress tells the backing loadbalancer to route requests based on the Host header.

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
![Name based virtual hosting.png](_images/Name based virtual hosting.png)
Default Backends: An Ingress with no rules, like the one shown in the previous section, sends all traffic to a single default backend. You can use the same technique to tell a loadbalancer where to find your website’s 404 page, by specifying a set of rules and a default backend. Traffic is routed to your default backend if none of the Hosts in your Ingress match the Host in the request header, and/or none of the paths match the url of the request.

**Loadbalancing:**
An Ingress controller is bootstrapped with some loadbalancing policy settings that it applies to all Ingress, such as the loadbalancing algorithm, backend weight scheme etc. More advanced loadbalancing concepts (eg: persistent sessions, dynamic weights) are not yet exposed through the Ingress. You can still get these features through the service loadbalancer. With time, we plan to distill loadbalancing patterns that are applicable cross platform into the Ingress resource.

It’s also worth noting that even though health checks are not exposed directly through the Ingress, there exist parallel concepts in Kubernetes such as readiness probes which allow you to achieve the same end result.

![Loadbalancer.png](_images/Loadbalancer.png)

You can set a service to be of type LoadBalancer the same way you’d set NodePort— specify the type property in the service’s YAML. There needs to be some external load balancer functionality in the cluster, typically implemented by a cloud provider.

This is typically heavily dependent on the cloud provider—GKE creates a Network Load Balancer with an IP address that you can use to access your service.

Every time you want to expose a service to the outside world, you have to create a new LoadBalancer and get an IP address.
Updating an Ingress:
Say you’d like to add a new Host to an existing Ingress, you can update it by editing the resource:

![Updating Ingress.png](_images/Updating Ingress.png)

This should pop up an editor with the existing yaml, modify it to include the new Host.

spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
          serviceName: s1
          servicePort: 80
        path: /foo
  - host: bar.baz.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
        path: /foo
saving it will update the resource in the API server, which should tell the Ingress controller to reconfigure the loadbalancer.

![Ingress-Output.png](_images/Ingress-Output.png)
You can achieve the same by invoking kubectl replace -f on a modified Ingress yaml file.

### **Setting Up an Ingress Controller on a Cluster**
You can set up different open source ingress controllers on clusters you have created with Container Engine for Kubernetes.

This topic explains how to set up an example ingress controller along with corresponding access control on an existing cluster. Having set up the ingress controller, this topic describes how to use the ingress controller with an example hello-world backend, and how to verify the ingress controller is working as expected.

**Example Components**
The example includes an ingress controller and a hello-world backend.

**Ingress Controller Components**
**The ingress controller comprises:**

An ingress controller deployment called nginx-ingress-controller. The deployment deploys an image that contains the binary for the ingress controller and Nginx. The binary manipulates and reloads the /etc/nginx/nginx.conf configuration file when an ingress is created in Kubernetes. Nginx upstreams point to services that match specified selectors.
An ingress controller service called ingress-nginx. The service exposes the ingress controller deployment as a LoadBalancer type service. Because Container Engine for Kubernetes uses an Oracle Cloud Infrastructure integration/cloud-provider, a load balancer will be dynamically created with the correct nodes configured as a backend set.

**Backend Components**
The hello-world backend comprises:

A backend deployment called docker-hello-world. The deployment handles default routes for health checks and 404 responses. This is done by using a stock hello-world image that serves the minimum required routes for a default backend.
A backend service called docker-hello-world-svc.The service exposes the backend deployment for consumption by the ingress controller deployment.

Setting Up the Example Ingress Controller
In this section, you create the access rules for ingress. You then create the example ingress controller components, and confirm they are running.

**Creating the Access Rules for the Ingress Controller**

1-If you haven't already done so, follow the steps to set up the cluster's kubeconfig configuration file and (if necessary) set the KUBECONFIG environment variable to point to the file. Note that you must set up your own kubeconfig file. You cannot access a cluster using a kubeconfig file that a different user set up. See Setting Up Cluster Access.
2-If your Oracle Cloud Infrastructure user is a tenancy administrator, skip the next step and go straight to Creating the Service Account, and the Ingress Controller.
3-If your Oracle Cloud Infrastructure user is not a tenancy administrator, in a terminal window, grant the user the Kubernetes RBAC cluster-admin clusterrole on the cluster by entering:
kubectl create clusterrolebinding <my-cluster-admin-binding> --clusterrole=cluster-admin --user=<user-OCID>
where:
<my-cluster-admin-binding> is a string of your choice to be used as the name for the binding between the user and the Kubernetes RBAC cluster-admin clusterrole. For example, jdoe_clst_adm
<user-OCID> is the user's OCID (obtained from the Console ). For example, ocid1.user.oc1..aaaaa...zutq (abbreviated for readability).

**For example:**
kubectl create clusterrolebinding jdoe_clst_adm --clusterrole=cluster-admin --user=ocid1.user.oc1..aaaaa...zutq
Creating the Service Account, and the Ingress Controller
Run the following command to create the nginx-ingress-controller ingress controller deployment, along with the Kubernetes RBAC roles and bindings:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
Create and save the file cloud-generic.yaml containing the following code to define the ingress-nginx ingress controller service as a load balancer service:

kind: Service
apiVersion: v1
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  ports:
    - name: http
      port: 80
      targetPort: http
    - name: https
      port: 443
      targetPort: https
Using the file you just saved, create the ingress-nginx ingress controller service by running the following command:

kubectl apply -f cloud-generic.yaml
Verifying the ingress-nginx Ingress Controller Service is Running as a Load Balancer Service
View the list of running services by entering:

kubectl get svc -n ingress-nginx
The output from the above command shows the services that are running:


NAME            TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                       AGE
ingress-nginx   LoadBalancer   10.96.229.38   <pending>      80:30756/TCP,443:30118/TCP    1h
The EXTERNAL-IP for the ingress-nginx ingress controller service is shown as <pending> until the load balancer has been fully created in Oracle Cloud Infrastructure.

Repeat the kubectl get svc command until an EXTERNAL-IP is shown for the ingress-nginx ingress controller service:

kubectl get svc -n ingress-nginx
The output from the above command shows the EXTERNAL-IP for the ingress-nginx ingress controller service:


NAME            TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)                       AGE
ingress-nginx   LoadBalancer   10.96.229.38   129.146.214.219   80:30756/TCP,443:30118/TCP    1h
Creating the TLS Secret
A TLS secret is used for SSL termination on the ingress controller.

Output a new key to a file. For example, by entering:

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=nginxsvc/O=nginxsvc"
To generate the secret for this example, a self-signed certificate is used. While this is okay for testing, for production, use a certificate signed by a Certificate Authority.

 Note

Under Windows, you may need to replace "/CN=nginxsvc/O=nginxsvc" with "//CN=nginxsvc\O=nginxsvc" . For example, this is necessary if you run the openssl command from a Git Bash shell.
Create the TLS secret by entering:

kubectl create secret tls tls-secret --key tls.key --cert tls.crt
Setting Up the Example Backend
In this section, you define a hello-world backend service and deployment.

Creating the docker-hello-world Service Definition
Create the file hello-world-ingress.yaml containing the following code. This code uses a publicly available hello-world image from Docker Hub. You can substitute another image of your choice that can be run in a similar manner.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-hello-world
  labels:
    app: docker-hello-world
spec:
  selector:
    matchLabels:
      app: docker-hello-world
  replicas: 3
  template:
    metadata:
      labels:
        app: docker-hello-world
    spec:
      containers:
      - name: docker-hello-world
        image: scottsbaldwin/docker-hello-world:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: docker-hello-world-svc
spec:
  selector:
    app: docker-hello-world
  ports:
    - port: 8088
      targetPort: 80
  type: ClusterIP
Note the docker-hello-world service's type is ClusterIP, rather than LoadBalancer, because this service will be proxied by the ingress-nginx ingress controller service. The docker-hello-world service does not need public access directly to it. Instead, the public access will be routed from the load balancer to the ingress controller, and from the ingress controller to the upstream service.

Deploy a sample Hello App using the below commands 

kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:1.0

Kubectl get pod

Pods are deployed in default namespace

Once the hello-app deployed expose the app to connect from cluster from port 8080 on default namespace

kubectl expose deployment hello-app  --port=8080

Kubectl get svc

Service deployed on default namespace

Deploy ingress controller using node port service to connect from outside cluster

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/baremetal/deploy.yaml

Ingress controller will deployed on ingress-nginx namespace

Kubectl get pods -n ingress-nginx

Kubectl get svc -n ingress-nginx


Sample hello-app ingress example.

cat << EOF | kubectl apply -f -   
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: hello-app
spec:
  rules:
  - http:
      paths:
      - path: /hello
        backend:
          serviceName: hello-app
          servicePort: 8080
EOF

Hello app ingress is creating on default namespace.

Kubectl get ing

It will display the ingress address to connect from browser

We are using node port service so we can able to access the site using node port of the ingress controller which we can get it from 

Kubectl get svc -n ingress-nginx  eg: 80:31688/TCP,443:3118

http://<EXTERNAL_IP>:31688/hello

**NGINX Installation Guide:**

ingress-nginx can be used for many use cases, inside various cloud provider and supports a lot of configurations. In this section you can find a common usage scenario where a single load balancer powered by ingress-nginx will route traffic to 2 different HTTP backend services based on the host name.

First of all follow the instructions to install ingress-nginx. Then imagine that you need to expose 2 HTTP services already installed: myServiceA, myServiceB. Let's say that you want to expose the first at myServiceA.foo.org and the second at myServiceB.foo.org. One possible solution is to create two ingress resources:

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-myservicea
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: myservicea.foo.org
    http:
      paths:
      - path: /
        backend:
          serviceName: myservicea
          servicePort: 80
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-myserviceb
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: myserviceb.foo.org
    http:
      paths:
      - path: /
        backend:
          serviceName: myserviceb
          servicePort: 80

When you apply this yaml, 2 ingress resources will be created managed by the ingress-nginx instance. Nginx is configured to automatically discover all ingress with the kubernetes.io/ingress.class: "nginx" annotation. Please note that the ingress resource should be placed inside the same namespace of the backend resource.

On many cloud providers ingress-nginx will also create the corresponding Load Balancer resource. All you have to do is get the external IP and add a DNS A record inside your DNS provider that point myServiceA.foo.org and myServiceB.foo.org to the nginx external IP. Get the external IP by running:

**Install NGINX Controller:**

Attention: The default configuration watches Ingress object from all namespaces.

To change this behavior use the flag --watch-namespace to limit the scope to a particular namespace.

warning : If multiple Ingresses define paths for the same host, the ingress controller merges the definitions.

The first time the ingress controller starts, two Jobs create the SSL Certificate used by the admission webhook. For this reason, there is an initial delay of up to two minutes until it is possible to create and validate Ingress definitions.

You can wait until it is ready to run the next command:

kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s

**Provider Specific Steps**
Run command
minikube addons enable ingress

**TLS TERMINATION IN AWS LOAD BALANCER (ELB)**
In some scenarios is required to terminate TLS in the Load Balancer and not in the ingress controller.

wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/aws/deploy-tls-termination.yaml
Edit the file and change:

VPC CIDR in use for the Kubernetes cluster:

proxy-real-ip-cidr: XXX.XXX.XXX/XX

**Deploy the manifest:**
kubectl apply -f deploy-tls-termination.yaml

Idle timeout value for TCP flows is 350 seconds and cannot be modified.

For this reason, you need to ensure the keepalive_timeout value is configured less than 350 seconds to work as expected.

Note: For private clusters, you will need to either add an additional firewall rule that allows master nodes access to port 8443/tcp on worker nodes, or change the existing rule that allows access to ports 80/tcp, 443/tcp and 10254/tcp to also allow access to port 8443/tcp.
Run this command:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/cloud/deploy.yaml

**Bare-metal**
Using NodePort:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/baremetal/deploy.yaml

Tip:
Applicable on kubernetes clusters deployed on bare-metal with generic Linux distro(Such as CentOs, Ubuntu ...).

**Verify installation:**

In minikube the ingress addon is installed in the namespace kube-system instead of ingress-nginx
Command:
kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx --watch

Once the ingress controller pods are running, you can cancel the command typing Ctrl+C.

Now, you are ready to create your first ingress.

**Detect installed version:**
To detect which version of the ingress controller is running, exec into the pod and run 

nginx-ingress-controller --version

**Using Helm**
NGINX Ingress controller can be installed via Helm using the chart from the project repository. To install the chart with the release name ingress-nginx
Coomands:

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx

**Detect installed version**

POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -- /nginx-ingress-controller --version

### what happens during an update

So what can you expect from your application’s availability during your cloud version upgrade?
Generally, you can expect the following three stages for your containerized application during an update:
* Preparing the cluster for upgrade.
* Upgrading the cluster core components.
* Upgrading the cluster add-on components.
In the first stage of preparing the cluster for upgrade, the cluster data is normally backed up before the real upgrading process begins. In the second stage, core components will upgrade to a newer version. Using IBM Cloud Private as an example, components like apiserver, controller-manager, and scheduler will upgrade to a newer version. Generally, applications won't call the core components directly, so the first two stages won't affect your applications!
In the third stage, add-on components like Calico, KubeDNS, and the NGINX Ingress Controller are upgrading, as your applications rely on these components, so you can expect there will be some (minimal) outage during the add-on components upgrade portion.
Tips to implement ahead of an update
There are four places where outages can occur during an update.
1. **Container**
Besides the 3 upgrade stages I mentioned, you might also want to upgrade the container version, like Docker. Upgrading Docker will restart all the containers that are running on the host. This will affect your application availability and this is an outage that is hard to avoid.
2. **Container network**
Pod-to-pod communication depends on the stability of container network. Upgrading network components might affect your container network. The stability of the container network depends on your cloud cluster. Using IBM Cloud Private as an example, IBM Cloud Private uses Calico as the default Container Network Interface (CNI) plug-in, and there is no downtime in the container network during upgrade.
3. **DNS**
As the typical containerized application shows, pod internal communication depends on the cluster domain name service. To reach a zero downtime upgrade, DNS must achieve both a graceful shutdown and a rolling upgrade. Graceful shutdown ensures the processing request is finished before pods exist. For a rolling upgrade request, you need to have at least two pods in your instance and ensure that there is always a pod available during upgrade.
If the upgrade can’t achieve graceful shutdown and rolling upgrade, the outage can last a few seconds during a DNS upgrade. During the outage, internal calls to the pod by service name might fail.


