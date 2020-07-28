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