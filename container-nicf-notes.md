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

The source code should expose some interface for these to be read. 

### Lifecycle Conformance Principle

Similar to above, image should have defined way to receive events from the
runtime. For example, it should handle SIGTERM and SIGKILL.