# Kubernetes 101 Training Session

<https://github.com/mfamador/k8s-demo>

## Orchestrators

- Manage networking and access
- Track state of containers
- Scale services
- Load balancing
- Relocation in case of unresponsive host
- Service discovery
- Attribute storage to containers
- Scale cluster
- Self healing services

## Kubernetes

- Greek for governor, helmsman, captain
- Open source container orchestration system
- Originally designed by Google (inspired by Borg), maintained by CNCF
- Aims to provide *platform for automating deployment, scaling and operations of application containers across clusters of hosts*

### Why?

- Allow scaling of both software and teams
  - Declarative configuration - declare desired state and Kubernetes' job is to ensure it matches
  - Self-healing systems - trying to mantain desired states if something changes
- Present abstract infrastructure
  - Decoupling container images and machines
  - Cluster can be heterogeneous and reduce overhead and cost
- Gain efficiency
  - Optimized usage of physical machines - multiple containers on same machine
  - Isolated with namespaces, to not interfere with each other

## Architecture

- Control Plane
  - `etcd` - consitent and highly-available key value store used as backing store for all cluster data
  - `kupe-apiserver` - exposes the k8s API. It is the front-end for the Kubernetes control plane
  - `kube-controller-mananager` - runs controllers processes
  - `kube-schedule` - watches newly created pods that have no node assigned and selects a node for them to run on

- Nodes
  - `kubelet` - agent which makes sure that containers are running in a pod
  - `kube-proxy` - network proxy - implementing part of the Service concept
  - `container runtime` - software that is responsible for running containers

## Resources and custom resources (CRDs)

- Workloads
  - `Pod (po)` - The basic deployable unit containing one or more processes in co-located containers
  - `ReplicaSet (rs)` - Keeps on or more pod replicas running
  - `Job` - Runs pods that perform a completable task
  - `CronJob` - Runs a scheduled job once or periodically
  - `DaemonSet (ds)` - Runs one pod replica per node (on all nodes or only on those matching a node selector)
  - `StatefulSet (sts)` - Runs stateful pods with a stable identity
  - `Deployment (deploy)` - Declarative deployment and updates of pods
- Services
  - `Service (svc)` - Exposes one or more pods at a single and stable IP address and port pair
  - `Endpoints (ep)` - Defines which pods (or other servers) are exposed through a service
  - `Ingress (ing)` - Exposes one or more services to external clients through a single externally reachable IP address
- Config
  - `ConfigMap (cm)` - A key-value map for storing non-sensitive config options for apps and exposing it to them
  - `Secret` - Like a ConfigMap, but for sensitive data
- Storage
  - `PersistentVolume* (pv)` - Points to persistent storage that can be mounted into a pod through a PersistentVolumeClaim
  - `PersistentVolumeClaim (pvc)` - A request for and claim to a PersistentVolume
  - `StorageClass* (sc)` - Defines the type of dynamically-provisioned storage claimable in a PersistentVolumeClaim

_* cluster-level resource (nor namespaced)_

### Pod

Pod is the smallest deployable unit of computing, not a container

Each container runs in its own cgroup (CPU+RAM allocation), but they share some namespaces and filesystems, such as:
    - IP address and port space
    - same hostname

#### When to have multiple container inside a pod?

- when it's impossible for them to work on different machines (sharing local filesystem or using IPC)
- when one of them facilitates communication with the other without altering it (adapter)
- when one of them offers support for the other (logging/monitoring/tracing/circuit breaking, e.g service mesh)
- when one of them configures the other

### Pod health checks

- `startupProbe` - indicates wether the application within the Container is started. All other probes are disabled if a startup probe is provided, until it succeeds
- `liveness prob` - application specific, determines if application does what the probe knows it should do
- `readiness probe` - on start, it might take a while until the application fully loads and can process requests as expected
  
### Pod affinity and anti-affinity

`nodeSelector` provides a very simple way to constrain pods to nodes with labels and the affinity/anti-affinity feature, greatly expands the types of constraints you can express.

### Deployment

A Deployment controller provides declarative updates for Pods and ReplicaSets.  
You describe a desired state in a Deployment, and the controller changes the actual state to the desired state at a controller rate. You can define Deployments and adopt all their resources with new Deployments.

### ReplicaSet

A ReplicaSet's purpose is to mantain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

### DaemonSet

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

### Service discovery

- find which processes are listening at which addresses for which services
- do it quickly and reliably, with low-latency, storing richer definitions of what those services are
- public DNS isn't dynamic enough to deal with the amount of updates

#### Communication challenges

- between pods - using hardcoded IPs would be the wrong way to do it, as pods might be rescheduled on different nodes and change IPs
- from outside - keep track of all pods that provide a certain service and load balance between them

### Service

Service is an abstraction which defines a logical set of Pods (selected using label selector), that provide the same functionality (same microservice)

Different types, for different types of exposure provided by the service:

- ClusterIP
- NodePorts
- LoadBalancer
- None

### ConfigMap and Secrets

ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized application portable.  
The ConfigMap resource provides a way to inject configuration data into Pods. The data stored in a ConfigMap object can be referenced in a volume of type configMap and then consumed by containerized applications running in a Pod.

Secrets let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys.

### Jobs

A Job creates one or more Pods and ensures that a specified number of them successfuly terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it crated.

### Volumes

On-disk files in a Container are ephemeral, which presents some problems for non-trivial applications when runnning in Containers.

First, when a Container crashes, kubelet will restart it, but the files will be lost - the Container starts with a clean state.

Second, when running Containers together in a Pod it is often necessary to share file between those Containers.

The Kubernetes Volume abstraction solves both problems.

A PersistenteVolume is a piece of storage in the cluster that has been provisioned manually or dynamically provisioned using Storage Classes.

A PersistenteVolumeClaim is a request for storage by user.

### StatefulSet

StatefulSet is the workload API object used to manage stateful applications.  
Manages the deployment and scaling of a set of Pods and provides guarantees about the oreding and uniqueness of these Pods.  
Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec but are not interchangeable: each has a persistent identifier that it mantains across any rescheduling.

StatefulSets are valuable for applications that require one or more of the following:

- Stable, unique network identifiers
- Stable, persistent storage
- Ordered, graceful deployment and scaling
- Ordered, automated rolling updates

### Ingress

Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.

For the Ingress resource to work, the cluster must have an ingress controller running.

### Service Mesh

- Code Independent
- Intelligent Routing and Load-Balancing
  - Canary Releases
  - Dark Launches
- Distributed Tracing
- Circuit Breakers
- Fine gained Access Control
- Telemetry, metrics and Logs
- Fleet wide policy enforcement

## Networking

### Fundamentals

- All Pods can communicate with all other Pods without NAT
- All nodes can communicate with all Pods (and vice-versa) without NAT
- The IP that a Pod sees itself as is the same IP that others see it as
- Containers in a pod exist within the same network namespace and share an IP (allowing for intrapod communication over localhost)
- Pods are given a cluster unique IP for the duration of its lifecycle, but the pods themselves are fundamentally ephemeral
- Services are given a persistent cluster unique IP that spans the Pods lifecycle
- External Connectivity is generally handed by an integrated cloud provider or other external entity (load balancer)
- Networking within Kubernetes is plumbed via the Container Network Interface (CNI), an interface between a container runtime and a network implementation plugin. (kubenet, Azure CNI, AWS CNI, Weave-net, Calico, Flannel, ...)

## Controllers and Operators

### Operators

Operators are software extensions to Kubernetes that make use of custom resources to manage appliactions and their components. They consist of one or more Custom Resource Definitions (CRDs) and a controller running inside the cluster.

- pre and post provision hooks, for appliaction-specific operations
- single tool to perform all management (kubectl)
- work in a scalable, repeatable, standard fashion
- improve resiliency while reducing burden on IT teams

## Helm

- Package Manager for Kubernetes
- Provides higher-level abstraction (Chart) to configure full-fledged applications
- Manage complexity, easy upgrades, simple sharing of full application setups, safe rollbacks

A chart is a collection of files that describe a related set of Kubernetes resources. A single chart might be used to deploy something simple, like a memcached pod, or something complex, like a full web app stack with HTTP servers, databases, caches, and so on.

## GitOps Workflow

### Continuous Delivery / Deployment

GitOps is a way to do Kubernetes cluster management and application delivery. It works by using Git as a single source of truth for declarative infrastructure and appliactions. With Git at the center of your delivery pipelines, developers can make pull requests to accelerate and simplify application deployments and operations tasks to Kubernetes.

## Useful Tools

- kubectx /kubens - Change context and default namespace quickly
- Stern - tail logs
- Grafana dashboards - Monitor
- Lens / k9s - Monitor and manage k8s clusters
- Kube Forwarder - Port forwarder
- Grafana loki - Logs

## Demo

- Start local K8s cluster
- Create a deployment with replicas
- Scale up the deployment
- Rolling upgrade deployment
- Create a service and check pod logs
- Create a configmap
- Use the configmap in a deployment
- Create a secret
- Use the secret in a deployment
- Create a job and a cronjob
- Create a daemonset and service
- Install and see K8s dashboard

### Kubernetes Hands-On Training Session

[Demo](https://github.com/davidandradeduarte/k8s-demo)
