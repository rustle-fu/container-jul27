# NUS-ISS NICF Container Course

## Day 1

A weakness of traditional applications is that they dictate their own resource 
usage. For example, a node application is always single threaded, so it won't
scale well on a multi-core machine. 

Containers are an ideal way to deliver a *microservices* architecture. The
services of the application are broken up into individual containers that can be
replicated and dristributed as needed. 

To refactor a monolith, break it up into *contexts* connected by interfaces.
A context cannot alter the data of another context.

Communication between microservices:
- Transport binding: HTTP, HTTP2, TCP/IP
- Security binding: tokens (JWT)
- Discovery: DNS
- Service description: WSDL SOAP, Open API
- Messaging protocols: req/res, one-way, subscription based

*Transactions* are easy in a monolith but hard in microservices. This arises
from the separation of data into contexts. 

### Scaling

#### Cloning

Run multiple instances of the application in parallel. Typically used with
monoliths. 

The database typically remains as a single instance.

#### Sharding

Partition the data across instances. A gateway determines which shard requests
go to.

#### Functional decomposition

Break up the application into microservices, each serving one function. 

### 12 Factor Application

Principles for scalable microservice apps. 

1. Codebase: 1 codebase, 1 app. Tracked by version control.
2. Dependencies: declare and isolate
3. Configurations: store in environment (config files etc.)
4. Backing services: treat as resources. Connection details in config. Allows
   uniform failure handling
5. Build, Release, Run: strictly separate stages and environments
6. Processes: Externalise all data to backing store; app should be stateless
   processes. Session cookies are the OPPOSITE of this.
7. Port Binding: avoids port crowding. 
8. Concurrency: Add more processes to scale (Statelessness enables this)
9. Disposability: Fast startup, graceful shutdown
10. Dev/Prod Parity: Dev, staging, prod envs should be as close as possible
11. Logs: As event stream/time series.
12. Admin processes: include as one-off processes.

### Containers and Docker

Virtualisation types:

Type 1: Hypervisor (aka VMM) runs directly on hardware
Type 2: Hypervisor runs on OS, which runs on hardware

> Despite being a "Windows" feature, Hyper-V is a type 1 hypervisor!

On top of the hypervisor, we have the VM, which in turn supports the OS and
applications on top of it.

In contrast, containers replace the hypervisor with Linux Containers (LXC). The
layers are:

1. App
2. Namespace
3. LXC
4. Linux
5. Hardware

> A container virtualises Linux.

It is the **namespace** that provides isolation and virtualises Linux resources.
**CGroups** allocates resources (with soft and hard limits) to containers.

### DigitalOcean Example

We use DigitalOcean's service to provision compute resources.

We create a personal access token on DO then use `docker-machine` with the DO
driver to create the machine. 

DO requires the machine size, region, OS and token to create the machine.

We create a machine with Docker engine installed on DO. Then we configure our
local docker environment to point to that machine, allowing control.

Run the following:
```
docker-machine env mymachine
```
This will output a command that configures your local docker variables for that
terminal session. 

>Normally, you would update a machine after creating it (to the specs you want).

### Images and Containers

An image is a packaged application. A container is an *instance* of said
application. 

### Packaging and Deploying an app

Suppose you have a node JS project. The steps you would take to get it up and
running are:

0. Install node
1. Have the files (code etc.)
2. Install dependencies (needs `npm`!)
3. Run (needs `node`!)

These instructions are written into the **Dockerfile**, which can be split into
the building stage and running stage. 

### Port Binding

The image's source code will typically control it's exposed port. So, to deploy
multiple instances, we use port binding/mapping to set an host's port that
maps to the port known to the code. The image is immutable, so it's own port
should not change. We set this binding during `docker run`

### Accessing a Container

`docker exec -ti container-name /bin/bash`

Lets you access the running container. Good for gathering information and
debugging. But actual fixes should be done outside and the image rebuilt. 

### High Observability Principle

To monitor containers, it is recommended to have at least the following
observables:

- Rediness
- Liveness
- Tracing
- Logs

The source code should expose some interface for these to be read. These can be
utilised by the HEALTHCHECK feature in Dockerfiles. 

### Lifecycle Conformance Principle

Similar to above, image should have defined way to receive events from the
runtime. For example, it should handle SIGTERM and SIGKILL.

### Containers and persistence

Containers are ephemeral; their data disappears when they are removed (or just
stopped?). To persist data, we can:

- Mount a local directory directly. We pass a command line option to docker run
  that gives the container access to a local folder, mounted at some point on
  the container. 
- Define a Docker **volume** and mount it to the container (using Dockerfile).
  Volumes allow for finer control over storage properties:
  - local vs remote
  - storage type (block/file/object)

`docker inspect image` returns a lot of information about the image. This can be
used to check things like volumes, entrypoint and so on. 

### Docker networks

We can use networks to let containers communicate and share resources (when
appropriate). For example we can have a DNS. The DNS allows containers to refer
to each other by names supplied as ENV in Dockerfiles. 

Docker has a default network (type bridge) named `bridge`. When a network is not
specified, this is what containers will connect to. 

Containers in a network will automatically use the network DNS, which
automatically builds the appropriate entries eg.

```
A <IP> container-1
A <IP> container-2
```
### Summary

- Nature of containers
- Containerising a node application
- Basic container networking

## Day 2

### Nginx

An Nginx container can serve as a reverse proxy. It helps to direct requests to
existing containers that might have a variety of external IPs. More on Day 3.

Nginx is very powerful; for simple applications *Caddy* will suffice as a file
server/reverse proxy.  

### Web Design Principle

Generally static resources are kept separately from logic. These are kept on a
resource server for greater speed and can even be cached. The logic is instead
on an application server (eg. Tomcat).

### Kubernetes

A container orchestration framework.

Use cases:
- Proper volume management (multinode redundancy)
- Graceful failover
- Graceful upgrade/patching
- Automatic scaling and load balancing

#### Logical organisation

A Kubernetes cluster is logically composed of objects called "Resources". These
resources are organised into *namespaces*, which partition the cluster.
Namespaces are good for handling "roles" - there is a default namespace called
`kube-system` that handles administration work (deleting it will break the
cluster). 

#### Architecture

Master-worker architecture. Master contains only control plane, no workload
(normally).  

Master:
- API Server (command and control)
- Scheduler (finds nodes for pods to run on)
- Controller (reconciles desired state with actual state)
- etcd (data store)

Worker:
- Containers in pods that run on Docker
- kubelet (talks to API server)
- kube-proxy (network stuff between pods and workers: config firewall, ports)

A load balancer will act as a gateway to route client requests to the workers.

> You should go for an odd number of nodes. This allows quorum in case of a split.

### DIY

Need to deploy a cluster? Can't use cloud? Consider the following:

- Kubernetes the hard way (manual, native)
- `Kubeadm` (containerised)

### Controlling Kubernetes

Native dashboard (not always provisioned). `kubectl` and other API/libraries are
available.

Kubernetes configuration is set in a YAML file. **This contains a secret
token**. In a managed service, this is generated for you. It should be stored in
a location analogous to `~/.kube` and be named `config` (no extension).
Alternatively, for a single session you can set the environment variable
`KUBECONFIG`.

> DO will refresh the config every 7 days.

This config is used by `kubectl` to find your cluster.

Kubernetes resources are sorted into namespaces. Use `kubectl get ns` to see
them. Resources can be specified in YAML and then created/destroyed by kubectl.
Other programmatic methods such as pulumi exist. 

### Pods

The smallest unit of work. Can contain multiple containers, but usually one. A
pod has an IP address (only accessible in the cluster), and internal containers
communicate on localhost. Pods are ephemeral and can move around. Hence, their
IP is not static.

### Deployments

A deployment is a Kubernetes resource that specifies a number of pods: their
replication, image and so on. Think of it as a pod management handle. It
specifies the number of replicas and the *template* for the pods.

The replica set resource is used by deployments to ensure the actual state
matches the desired state. 

### Accessing the application

As pods die and start, their IPs effectively change. To provide a stable IP for
clients to access the pods, we use `Services`. These basically handle access
control and load balancing.

Pods are associated with services if at least one of its pod labels match.

Unlike pods, services are durable (in terms of IP and namespace DNS name).

#### Service Types

These service types function slightly differently and work with each other. 

- ClusterIP: inside the Kubernetes cluster, inaccessible from outside. 
- NodePort: Opens a port on all nodes which maps to the ClusterIP
- LoadBalancer: accessible from outside

Traffic coming from outside will go LoadBalancer -> NodePort -> ClusterIP.
A created LoadBalancer will contain the other two in it; a NodePort will contain
a ClusterIP. 

You can create them independently. For example, you can have a LoadBalancer to
handle client requests, and a ClusterIP that handles internal requests. Remember
that a cluster will have many kinds of Deployment, not all should be exposed.

### Summary

- Theoretical: Pods, Deployments, Services, Namespaces
- Practical: Connecting Deployments via services and configs
- kubectl YAML API
  - Variable injection and referencing
  - YAML resources are assessed in order

## Day 3

### Helm

`Helm` is a package manager that can ease deployment of Kubernetes applications.
Without it, we would have a mess of YAML files. Helm packages are known as
*Charts*. Charts will be on Helm Hub.

Think of the steps we took to build a containerised application using a
Kubernetes cluster. Those steps can be automated by `helm`.

You can get the YAML that describes a Chart Kubernetes application using 
`helm template`. 

You can also supply a YAML file to a chart to provide values such as DB names.

`spec.template.spec.

When we use a chart (installing nginx-ingress, Operators) it is good practice to
keep it in a namespace just for it. 

### Kubernetes Rolling Updates

Downtime free updating! Gradually replaces old pods (when they aren't serving
anything). A forced deployment would be `recreate`. You can specify the number
of new version pods brought in at a time pods `maxSurge` and the number of old
pods removed at a time `maxUnavailable`. These can also be set in terms of
percentage. 

An update is specified in the Deployment resource, under `spec.strategy`.

### Retrieving YAML from a live resource

The API server will save a copy of each Resource's YAML. To get it back use

```
kubectl get <resource> -o yaml
```

The one from the server will have other fields populated by default values; it
is a full spec.

### Version Management

```
kubectl rollout history <deployment> -n ns
kubectl rollout undo <deployment> -n ns --to-revision=<revision number>
```

### Health Checks

- Readiness
- Liveness

Specified in containers.readinessProbe and containers.livenessProbe.

### Storage and Kubernetes

Kubernetes Volumes are tied to Pods and thus ephemeral unlike Docker volumes.
They are specified in the pod's YAML or injected via ConfigMap

For persistent storage, it can be set manually (static) or dynamically.
Information about the storage policy is held in the *storage class*.

*Persistent Volume Claims* are resources used to link pod volumes to persistent
storage. 

Persistent volumes are seen as Kubernetes Volumes by the pod, but don't
disappear when the pod dies. If multiple pods talk to a persistent volume, it
must be able to handle multiple writes. 

> Declaring a PVC will result in a PV being created to match it (conditional on
> available resources).

### Admission Controllers

Control the assignment of resources. Deployments send requests here.

- Validating: checks the deployment and enforces policy
- Mutating: Changes the deployment (istio sidecar)

Admission controllers can read *annotations* which provide additional
information for requesting custom resources. 

`initContainers` are used Containers that are started before the others and
clean up the environment. They may be necessary to clean persistent volumes.
These initial containers must exit; otherwise the rest will not start up.

### Ingress

Load balancers are typically overkill, being provisioned by the underlying cloud
platform. We use an ingress, which is simpler and cheaper than a load balancer.
An ingress is a Kubernetes resource (not a Service) Nginx Ingress is a popular
Ingress Controller. 

We can set up an Ingress (set of rules) in and point it to a ClusterIP service. 

> source: https://stackoverflow.com/questions/45079988/ingress-vs-load-balancer

Load Balancer: A kubernetes LoadBalancer service is a service that points to
**external** load balancers that are NOT in your kubernetes cluster, but exist
elsewhere. They can work with your pods, assuming that your pods are externally
routable. Google and AWS provide this capability natively. In terms of Amazon,
this maps directly with ELB and kubernetes when running in AWS can automatically
provision and configure an ELB instance for each LoadBalancer service deployed.

Ingress: An ingress is really just a set of rules to pass to a controller that
is listening for them. You can deploy a bunch of ingress rules, but nothing will
happen unless you have a controller that can process them. A LoadBalancer
service could listen for ingress rules, if it is configured to do so.

You can also create a NodePort service, which has an externally routable IP
outside the cluster, but points to a pod that exists within your cluster. This
could be an Ingress Controller.

An Ingress Controller is simply a pod that is configured to interpret ingress
rules.

> We can use nip.io to fake domain names for testing. You can't use the IP
> directly because the request will have a different form (no host field) and
> the ingress will fail to understand. 

Routing can be complex depending on how your web app is organised. For more
details on how to arrange the ingress, check the notes.

### Scaling

Before we choose to scale, we should be measuring the cluster's performance. We
use Kubernetes metrics server. Once it is deployed it allows extra commands on
kubectl that monitor resource usage (`kubectl top`).

Using these gathered metrics, we can configure *automatic* scaling. We set up a
Horizontal Pod Autoscaler. It manages Deployments and overrrides the `replicas`
field. It does this by manipulating the Replica Set. 

### Tools

#### Cluster Monitoring Tools

Lensapp Lens (native), VMWare octant (web based). Alternatives to DO dashboard.

### Summary

- Health Checks (readiness/liveness)
- Helm package manager
- Storage
  - Volumes
  - Persistent volumes
  - Persistent volume claims
- Networking
  - Ingress (stable/nginx-ingress, Kong)
- Scaling
  - Requesting and limiting resources of Deployments
  - Horizontal Pod Autoscaler

## Day 4

### Operators

> See OperatorHub.

Operators can be installed via `helm`. They add custom resource types to the
cluster. They contain the expert knowledge for deploying applications, allowing
a declarative approach to configuring the application.

Operators thus simplify the deployment process. The are configured by Custom
Resource Configurations (CRD).

> Recall we helm installed a chart and supplied config YAML. Every time we
> deploy we would need to create a new release for (redo) it.
> Install Operator once so it can install charts for you.

Consider the presslabs mysql-operator Operator. It adds the MysqlCluster custom
resource that creates pods with 4 containers: mysql, sidecar, metrics-exporter,
and pt-heartbeat. Just by declaring one resource, we have a much more thorough
deployment.

### Stateful Applications

The typical deployment:
- Service entrypoint
- Ephemeral pods
  
is acceptable for stateless applications. However, the typical clustered
application has state, and master-servant relations between pod analogues. For
example, mysql has a master node and many read-only copies. They have unequal
roles, and the copies need to know who the master is. 

To describe this causal relationship, we can use a Statefulset. The MySQL
operator includes a statefulset. 

```
# Typical FQDN
servicename.namespacename.svc.cluster.local

# Headless service, managed by statefulset
podname.servicename.namespacename.svc.cluster.local
```

### Selectors and Labels

Certain resource specify Selectors. The selector selects resources which match
all of its defined labels (though the resource may have more labels of its own).

### DO Spaces

An S3 compatible object store designed to store and serve a large amount of
assets. 

### Istio

This is just a surface level lesson; Istio is much deeper.

Modern applications/services will often require certain infrastructure features.
Things like CORS, metering, authentication throttling and so on. There are 

- Horizontal features: Common to all services, transparent to the service.
  - Logging
  - Authentication
  - CORS
  - Fault management
  - Throttling

Horizontal features can be implemented on the load-balancer/ingress.

An abstraction of these common services can be implemented as a sidecar - a
container that is injected into each pod and proxies the communication across
the pod boundary. 

Istio is a **service mesh platform**. It can provide an API gateway (more
complex than Nginx) and **Envoy**-based sidecar proxies. It installs to the
`istio-system` namespace.

**Pilot** is part of Istio's control plane and pushes policy to the sidecar
proxies. 

Istio has several predefined profiles to help you get started. 

> `istioctl install --set profile=demo`

#### Sidecar feature example

Suppose a request comes in to Service 1 and is passed to Service 2, which has
failed. The sidecar will notice Service 2 is down and subsequent requests fail
immediately (circuit breaker).

Sidecars can be injected automatically on a per-namespace basis. This is enabled
by default, and may not be ideal for your application. You can opt-out via
annotation on a pod. Automatic injection requires a label on the namespace

> Recall that this injection is done by a mutating Admission Controller. 

Manual injection is aided by `istioctl kube-inject`. This outputs an altered
version of the supplied YAML file by injecting the sidecar to Deployments and
Pods declared in the YAML.

### Multiple Ingresses

When you have more than 1 ingress, you need to specify which ingress to which
you deploy an ingress resource.

- Annotations (deprecated recently, 1.17)
- ingressClassName

### Dashboards

Istio makes it easy to monitor via web dashboards.

> `istioctldash grafana | prometheus| jaeger | kiali`

- Jaeger: request traces
- Kiali: Kubernetes cluster info

### Summary

- Operators: Ready made, configurable deployments
- Istio: service mesh
  - Sidecars