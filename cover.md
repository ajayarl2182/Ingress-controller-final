# Ingress Controller

**What is Ingress?**

Kubernetes Ingress is an API object that provides routing rules to manage external users' access to the services in a Kubernetes cluster, typically via HTTPS/HTTP. ... Ingress allows you to configure and manage these capabilities inside the cluster.

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

![Ingress.png](_images/Ingress.png)

**An Ingress provides the following:**

1-Externally reachable URLs for applications deployed in Kubernetes clusters.

2-Name-based virtual host and URI-based routing support.

3-Load balancing rules and traffic, as well as SSL termination

**Ingress**

Ingress enables you to consolidate the traffic-routing rules into a single resource and runs as part of a Kubernetes cluster. Some reasons Kubernetes Ingress is the preferred option for exposing a service in a production environment include the following:

Traffic routing is controlled by rules defined on the Ingress Resource.
Ingress is part of the Kubernetes cluster and runs as pods.
An external Load Balancer is expensive, and you need to manage this outside the Kubernetes cluster. Kubernetes Ingress is managed from inside the cluster.


**What is the Ingress Controller?**

If Kubernetes Ingress is the API object that provides routing rules to manage external access to services, Ingress Controller is the actual implementation of the Ingress API. The Ingress Controller is usually a load balancer for routing external traffic to your Kubernetes cluster and is responsible for L4-L7 Network Services. 

Layer 4 (L4) refers to the connection level of the OSI network stack—external connections load-balanced in a round-robin manner across pods. Layer 7 (L7) refers to the application level of the OSI stack—external connections load-balanced across pods, based on requests. Layer 7 is often preferred, but you should select an Ingress Controller that meets your load balancing and routing requirements.

Ingress Controller is responsible for reading the Ingress Resource information and processing that data accordingly.

The following is a sample Ingress Resource:
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
spec:
  backend:
    serviceName: ServiceName
    servicePort: <Port Number>
  
Here is a simple example where an Ingress sends all its traffic to one Service:

An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.
An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type Service.Type=NodePort or Service.Type=LoadBalancer.


**Prerequisites:**

You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect.
You may need to deploy an Ingress controller such as ingress-nginx. You can choose from a number of Ingress controllers.
Ideally, all Ingress controllers should fit the reference specification. In reality, the various Ingress controllers operate slightly differently.

As with all other Kubernetes resources, an Ingress needs apiVersion, kind, and metadata fields. The name of an Ingress object must be a valid DNS subdomain name. For general information about working with config files, see deploying applications, configuring containers, managing resources. Ingress frequently uses annotations to configure some options depending on the Ingress controller, an example of which is the rewrite-target annotation. Different Ingress controller support different annotations. Review the documentation for your choice of Ingress controller to learn which annotations are supported.

The Ingress spec has all the information needed to configure a load balancer or proxy server. Most importantly, it contains a list of rules matched against all incoming requests. Ingress resource only supports rules for directing HTTP(S) traffic.



**What you will learn in this course**

1-How to create and manage Ingress Controller

2-Setting Up an Ingress Controller on a Kubernetes Cluster

3-what happens during an update
