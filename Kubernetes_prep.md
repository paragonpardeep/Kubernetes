## Kubernetes Prep


# Kubernetes Architecture — The "City + Restaurant" Mental Model

> **Goal:** Understand Kubernetes architecture so well that you can explain it on a whiteboard in 2–3 minutes and reason through production incidents without memorizing definitions.

---

## 1. The One Sentence You Must Remember

**Kubernetes is a control system that continuously makes the real world look like the state you declared.**

Think:

```text
YOU DECLARE WHAT YOU WANT
          ↓
   Kubernetes CONTROL
          ↓
Kubernetes makes it happen
          ↓
It keeps checking forever
          ↓
If reality changes → it fixes it
```

That is the heart of Kubernetes.

---

# 2. The Best Mental Model: A CITY

Imagine Kubernetes is a **huge smart city**.

| Kubernetes | City analogy | Easy meaning |
|---|---|---|
| Cluster | Entire city | Everything together |
| Control Plane | City administration | Makes decisions |
| API Server | City reception / front desk | Everyone talks through it |
| etcd | City master database | Remembers everything |
| Scheduler | Traffic/placement planner | Decides where workloads go |
| Controllers | City inspectors | Continuously fix differences |
| Worker Node | Building | Where applications actually run |
| Kubelet | Building manager | Manages workloads on that node |
| Container Runtime | Engine inside building | Actually starts containers |
| Pod | Apartment | Smallest deployable unit |
| Container | Person/business inside apartment | Application process |
| Service | Phone directory / stable address | Finds the right Pods |
| Ingress | Main city gate | Brings external HTTP/HTTPS traffic in |
| DNS | City directory | Converts names to addresses |
| ConfigMap | Public notice | Non-secret configuration |
| Secret | Locked safe | Sensitive configuration |
| Namespace | District | Logical separation |
| RBAC | Access badge system | Who can do what |
| HPA | Automatic building expansion | Adds/removes Pods |
| Cluster Autoscaler | Builds/removes buildings | Adds/removes nodes |

### 🧠 The memory trick

Remember just this:

> **Administration → Buildings → Apartments → People**

In Kubernetes:

> **Control Plane → Nodes → Pods → Containers**

Everything else fits around this.

---

# 3. The Complete Architecture

```text
                         KUBERNETES CLUSTER
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                    CONTROL PLANE                                   │
│              "THE CITY ADMINISTRATION"                             │
│                                                                     │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐       │
│   │ API SERVER   │────▶│    etcd      │     │  SCHEDULER   │       │
│   │ "FRONT DESK" │     │ "MEMORY"     │     │ "PLACEMENT"  │       │
│   └──────┬───────┘     └──────────────┘     └──────┬───────┘       │
│          │                                           │              │
│          │              ┌──────────────────┐         │              │
│          └─────────────▶│   CONTROLLERS    │◀────────┘              │
│                         │ "CITY INSPECTORS" │                       │
│                         └─────────┬────────┘                        │
│                                   │                                 │
│                         "MAKE REALITY MATCH DESIRED"                │
│                                   │                                 │
├───────────────────────────────────┼─────────────────────────────────┤
│                                   │                                 │
│                         WORKER NODES                               │
│                    "THE CITY BUILDINGS"                            │
│                                                                     │
│  ┌─────────────────────┐       ┌─────────────────────┐             │
│  │      NODE 1         │       │      NODE 2         │             │
│  │                     │       │                     │             │
│  │   ┌─────────────┐   │       │   ┌─────────────┐   │             │
│  │   │   KUBELET   │   │       │   │   KUBELET   │   │             │
│  │   │ "MANAGER"   │   │       │   │ "MANAGER"   │   │             │
│  │   └──────┬──────┘   │       │   └──────┬──────┘   │             │
│  │          │          │       │          │          │             │
│  │   ┌──────▼──────┐   │       │   ┌──────▼──────┐   │             │
│  │   │  CONTAINER  │   │       │   │  CONTAINER  │   │             │
│  │   │   RUNTIME    │   │       │   │   RUNTIME    │   │             │
│  │   └──────┬──────┘   │       │   └──────┬──────┘   │             │
│  │          │          │       │          │          │             │
│  │     ┌────▼────┐     │       │     ┌────▼────┐     │             │
│  │     │  POD A  │     │       │     │  POD B  │     │             │
│  │     └─────────┘     │       │     └─────────┘     │             │
│  └─────────────────────┘       └─────────────────────┘             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

# 4. The Most Important Concept: Desired State vs Actual State

This is the **secret behind Kubernetes**.

Suppose you tell Kubernetes:

```yaml
replicas: 5
```

You are saying:

> "I want 5 Pods."

Kubernetes observes:

```text
Desired State = 5
Actual State  = 5

        ✅
```

Now one Pod crashes:

```text
Desired State = 5
Actual State  = 4

        ❌
```

Kubernetes says:

> "Something is wrong."

A controller creates another Pod:

```text
Desired State = 5
Actual State  = 5

        ✅
```

### The Golden Rule

> **Kubernetes does not simply execute commands. It continuously reconciles state.**

Remember:

```text
DESIRED
   ↓
COMPARE
   ↓
ACTUAL
   ↓
DIFFERENCE?
   ↓
FIX IT
   ↓
CHECK AGAIN
   ↺
```

This is called the **control loop / reconciliation loop**.

---

# 5. API Server — The City Reception

The **kube-apiserver** is the most important entry point.

Almost everything talks through it.

```text
kubectl
   │
   ▼
API SERVER
   │
   ├──── Authentication
   ├──── Authorization
   ├──── Admission
   ├──── Validation
   │
   ▼
  etcd
```

Think:

> **API Server = Receptionist**

You don't walk directly into the city's database.

You go to the reception.

### Who talks to API Server?

```text
kubectl
Controllers
Scheduler
Kubelet
Operators
CI/CD
Monitoring tools
```

### Interview question

**Q: What happens when I run `kubectl get pods`?**

```text
kubectl
   ↓
API Server
   ↓
Authentication
   ↓
Authorization
   ↓
Read cluster state
   ↓
Response
   ↓
kubectl
```

---

# 6. etcd — Kubernetes' Memory

Think of **etcd as the city's master database**.

It stores Kubernetes cluster state.

Examples:

```text
Deployments
Pods
Services
ConfigMaps
Secrets
Nodes
RBAC objects
Namespaces
```

Important:

> **etcd stores Kubernetes state; it does NOT run your containers.**

### Memory trick

**etcd = "I remember everything about the cluster."**

---

# 7. Scheduler — The Apartment Allocator

Suppose a new Pod needs to run.

Who decides which Node gets it?

**Scheduler.**

```text
New Pod
   ↓
Scheduler
   ↓
Node 1? ❌
Node 2? ✅
Node 3? ❌
   ↓
Pod assigned to Node 2
```

Scheduler considers things such as:

- CPU/memory requests
- nodeSelector
- affinity
- anti-affinity
- taints/tolerations
- topology constraints
- resource availability
- priorities

### Important distinction

> **Scheduler decides WHERE.**

It does not actually start the container.

Remember:

**Scheduler = "Where should this Pod live?"**

---

# 8. Controllers — The City Inspectors

Controllers are the heart of Kubernetes' self-healing behavior.

They continuously ask:

> "Does reality match what the user requested?"

Example:

```text
Desired: 5 replicas
Actual:  4 replicas

Controller notices difference
          ↓
Creates another Pod
          ↓
Actual: 5
```

### Common controllers

- Deployment Controller
- ReplicaSet Controller
- StatefulSet Controller
- DaemonSet Controller
- Node Controller
- Job Controller
- EndpointSlice Controller

### Memory trick

> **Controllers don't sleep. They continuously reconcile.**

---

# 9. Worker Node — The Building

A Worker Node is where your workloads actually run.

It contains:

```text
NODE
│
├── kubelet
├── container runtime
├── networking components
└── Pods
```

Think:

> **Node = Building where applications live.**

---

# 10. Kubelet — The Building Manager

Kubelet is an agent running on every worker node.

The control plane tells it:

> "This node should run these Pods."

Kubelet makes sure those Pods actually exist and remain healthy according to the Pod specification.

```text
API Server
    ↓
Kubelet
    ↓
Container Runtime
    ↓
Container
```

### Memory trick

> **Kubelet = "Make sure the Pods assigned to my Node are running."**

---

# 11. Container Runtime — The Engine

Kubelet does not directly create containers.

It talks to the **container runtime** through the Container Runtime Interface (CRI).

Examples include:

- containerd
- CRI-O

Flow:

```text
Kubelet
   ↓
CRI
   ↓
Container Runtime
   ↓
Container
```

### Memory trick

> **Kubelet manages. Runtime runs.**

---

# 12. Pod — The Apartment

A Pod is the smallest deployable unit in Kubernetes.

Usually:

```text
Pod
└── Container
```

But a Pod can contain multiple tightly coupled containers:

```text
Pod
├── Application Container
└── Sidecar Container
```

Containers inside the same Pod share important resources such as:

- network namespace
- Pod IP
- localhost communication
- mounted volumes

### Important

> **Kubernetes schedules Pods, not individual containers.**

Remember:

> **Pod = the unit Kubernetes places on a Node.**

---

# 13. Service — The Stable Phone Number

Pods are temporary.

A Pod can die and be recreated with a different IP.

So how does another application find it?

**Service.**

```text
             SERVICE
          10.20.30.40
               │
       ┌───────┼───────┐
       ▼       ▼       ▼
     Pod A   Pod B   Pod C
```

Service provides a stable virtual endpoint and selects backend Pods.

### Memory trick

> **Pods change. Service stays.**

---

# 14. DNS — The Phone Book

Instead of remembering:

```text
10.20.30.40
```

Applications use names:

```text
payments.production.svc.cluster.local
```

CoreDNS provides Kubernetes service discovery.

```text
Application
     ↓
CoreDNS
     ↓
Service name
     ↓
Service IP
     ↓
Backend Pods
```

### Memory trick

> **DNS tells you the address. Service gets you to the Pods.**

---

# 15. Ingress — The Main City Gate

External users don't normally know individual Pod IPs.

Traffic enters through components such as:

```text
Internet
   ↓
Load Balancer
   ↓
Ingress
   ↓
Service
   ↓
Pods
```

Ingress provides HTTP/HTTPS routing rules.

Example:

```text
api.company.com
      ↓
Ingress
      ├── /users    → users-service
      ├── /orders   → orders-service
      └── /payment  → payment-service
```

### Memory trick

> **Ingress = Traffic Gate**

---

# 16. The Full Request Journey

This is one of the most useful things to memorize.

A user opens:

```text
https://api.company.com/orders
```

Think:

```text
                 USER
                   │
                   ▼
                 DNS
                   │
                   ▼
            Load Balancer
                   │
                   ▼
                Ingress
                   │
                   ▼
             Service
                   │
                   ▼
              Endpoint
                   │
                   ▼
                 Pod
                   │
                   ▼
             Container
                   │
                   ▼
              Application
```

### The 7-hop memory trick

> **Name → Gate → Route → Service → Pod → Container → App**

---

# 17. The Full Deployment Journey

Now memorize this one.

You run:

```bash
kubectl apply -f deployment.yaml
```

What happens?

```text
                 kubectl
                    │
                    ▼
               API SERVER
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        etcd              Admission/
       "store it"          validation

                    │
                    ▼
              DEPLOYMENT
               CONTROLLER
                    │
                    ▼
                ReplicaSet
                    │
                    ▼
              Scheduler
           "Choose a Node"
                    │
                    ▼
                Kubelet
                    │
                    ▼
           Container Runtime
                    │
                    ▼
                  Pod
                    │
                    ▼
               Container
```

### The magic sentence

> **API Server accepts it → etcd remembers it → Controller creates it → Scheduler places it → Kubelet manages it → Runtime runs it.**

If you remember this sentence, you can answer many Kubernetes architecture questions.

---

# 18. What Happens When a Pod Dies?

This is where Kubernetes becomes powerful.

```text
Pod A
  ↓
CRASH 💥
  ↓
Actual = 4
Desired = 5
  ↓
Controller notices
  ↓
Creates replacement Pod
  ↓
Scheduler chooses Node
  ↓
Kubelet starts Pod
  ↓
Actual = 5
```

### The key idea

> **Kubernetes doesn't "magically restart things." Controllers continuously reconcile desired state with actual state.**

---

# 19. What Happens When a Node Dies?

Suppose:

```text
Node 1 💥
```

Pods on that node become unavailable.

Then:

```text
Node failure
     ↓
Node Controller detects problem
     ↓
Workloads become unavailable
     ↓
Controllers maintain desired replicas
     ↓
New Pods are scheduled
     ↓
Healthy Nodes run replacements
```

Conceptually:

```text
NODE 1 💥

Pod A ❌
Pod B ❌
Pod C ❌

          ↓

Healthy Nodes

Node 2
Pod A ✅

Node 3
Pod B ✅
Pod C ✅
```

Exact behavior depends on workload type, eviction timing, storage, disruption rules, and cluster conditions.

---

# 20. Networking Architecture — The Easy Picture

Think of Kubernetes networking as **roads**.

```text
                 INTERNET
                     │
                     ▼
              Load Balancer
                     │
                     ▼
                  Ingress
                     │
                     ▼
                  Service
                     │
              ┌──────┴──────┐
              ▼             ▼
            Pod A         Pod B
              │             │
              └──────┬──────┘
                     │
                    CNI
                     │
                   Nodes
```

### Components to remember

**CNI**

> Creates/configures Pod networking.

**Service**

> Gives a stable virtual endpoint.

**CoreDNS**

> Resolves names.

**Ingress**

> Routes external HTTP/HTTPS traffic.

**NetworkPolicy**

> Controls allowed network communication.

---

# 21. Storage Architecture — The Easy Picture

Think of storage as:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
StorageClass
 ↓
CSI Driver
 ↓
Cloud / Physical Storage
```

Memory sentence:

> **Pod asks for storage through PVC → PVC gets storage from PV → StorageClass dynamically provisions it → CSI connects Kubernetes to the actual storage system.**

---

# 22. Security Architecture — The Easy Picture

Whenever somebody asks:

> "Can this user do this?"

Think:

```text
WHO ARE YOU?
     ↓
Authentication
     ↓
WHAT ARE YOU ALLOWED TO DO?
     ↓
Authorization / RBAC
     ↓
IS THE REQUEST ALLOWED BY POLICY?
     ↓
Admission
     ↓
API ACTION
```

Memory sentence:

> **Authentication = Who are you?**

> **Authorization = What can you do?**

> **Admission = Should this request be allowed into the cluster?**

---

# 23. Autoscaling — Two Different Things

This is a common interview trap.

### HPA

Changes:

```text
NUMBER OF PODS
```

```text
5 Pods
  ↓
10 Pods
```

### Cluster Autoscaler

Changes:

```text
NUMBER OF NODES
```

```text
10 Nodes
   ↓
15 Nodes
```

### Memory trick

> **HPA grows the workforce.**

> **Cluster Autoscaler grows the buildings.**

---

# 24. Kubernetes Architecture in One Picture

Memorize this:

```text
                         USERS / ENGINEERS
                                │
                                ▼
                         ┌─────────────┐
                         │ API SERVER  │
                         │ "FRONT DOOR"│
                         └──────┬──────┘
                                │
                ┌───────────────┼────────────────┐
                │               │                │
                ▼               ▼                ▼
             ┌─────┐       ┌──────────┐     ┌──────────┐
             │etcd │       │CONTROLLERS│     │SCHEDULER │
             │MEMORY│      │ RECONCILE │     │ WHERE?   │
             └─────┘       └─────┬────┘     └────┬─────┘
                                  │               │
                                  └───────┬───────┘
                                          ▼
                             ┌──────────────────────┐
                             │     WORKER NODE      │
                             │                      │
                             │  ┌───────────────┐   │
                             │  │    KUBELET    │   │
                             │  │   "MANAGER"   │   │
                             │  └───────┬───────┘   │
                             │          ▼           │
                             │  ┌───────────────┐   │
                             │  │    RUNTIME    │   │
                             │  │    "ENGINE"   │   │
                             │  └───────┬───────┘   │
                             │          ▼           │
                             │       ┌─────┐        │
                             │       │ POD │        │
                             │       └──┬──┘        │
                             │          ▼           │
                             │     CONTAINER        │
                             └──────────────────────┘
```

---

# 25. The 6 Words That Explain Kubernetes

If you forget everything else, remember:

```text
          DECLARE
             ↓
          STORE
             ↓
          WATCH
             ↓
          DECIDE
             ↓
          RUN
             ↓
          RECONCILE
             ↺
```

More concretely:

```text
DECLARE
"I want 5 Pods."

       ↓

STORE
"API Server stores the desired state."

       ↓

WATCH
"Controllers observe the cluster."

       ↓

DECIDE
"Scheduler decides where Pods go."

       ↓

RUN
"Kubelet + Runtime run them."

       ↓

RECONCILE
"Controllers continuously make reality = desired state."
```

---

# 26. FAANG-Level Interview: Explain It in 90 Seconds

If an interviewer says:

> **"Explain Kubernetes architecture."**

Don't start listing 15 components.

Say:

> "I think of Kubernetes as a distributed control system. The Control Plane maintains the desired state, while Worker Nodes run the workloads.
>
> The API Server is the front door for Kubernetes. Requests go through authentication, authorization and admission before the state is persisted in etcd.
>
> Controllers continuously watch the cluster and reconcile the actual state with the desired state. When a new Pod needs to run, the Scheduler determines which Node should host it based on resources and scheduling constraints.
>
> On the selected Node, kubelet is responsible for ensuring the Pod is running, and it communicates with the container runtime through CRI to start the containers.
>
> For application traffic, Services provide stable endpoints for dynamic Pods, CoreDNS provides service discovery, and Ingress or a Gateway can provide external HTTP routing.
>
> So the simplest flow is: API Server accepts the desired state, etcd stores it, controllers reconcile it, scheduler places Pods, kubelet manages them, and the runtime runs them."

That is a **much stronger answer** than simply naming components.

---

# 27. The Most Important Troubleshooting Trick

When something breaks, don't randomly run commands.

Follow the architecture.

## Example: Users get 503

Start from the outside:

```text
USER
 ↓
DNS
 ↓
LOAD BALANCER
 ↓
INGRESS
 ↓
SERVICE
 ↓
ENDPOINTS
 ↓
POD
 ↓
CONTAINER
 ↓
APPLICATION
 ↓
DEPENDENCY
```

Ask:

> "Where does the traffic stop?"

---

## Example: Pod is Pending

Start from scheduling:

```text
POD
 ↓
SCHEDULER
 ↓
NODE?
 ↓
CPU/MEMORY?
 ↓
TAINT?
 ↓
AFFINITY?
 ↓
TOPOLOGY?
 ↓
PVC?
 ↓
QUOTA?
```

---

## Example: Pod is CrashLoopBackOff

Start inside the workload:

```text
POD
 ↓
CONTAINER
 ↓
PROCESS
 ↓
EXIT CODE
 ↓
LOGS
 ↓
CONFIG
 ↓
SECRET
 ↓
PROBE
 ↓
RESOURCE
 ↓
DEPENDENCY
```

---

# 28. The Ultimate Memory Map

Draw this from memory before every Kubernetes interview:

```text
                     KUBERNETES
                         │
            ┌────────────┴────────────┐
            │                         │
       CONTROL PLANE              WORKERS
       "BRAIN"                    "MUSCLE"
            │                         │
      ┌─────┼─────┐             ┌────┼────┐
      │     │     │             │    │    │
     API   etcd Scheduler     Kubelet Runtime Pods
      │
 Controllers
      │
      └─────────── RECONCILE ───────────────┐
                                            │
                                            ▼
                                     DESIRED = ACTUAL


                    TRAFFIC
                       │
                      DNS
                       │
                 Load Balancer
                       │
                    Ingress
                       │
                    Service
                       │
                 EndpointSlice
                       │
                      Pod
                       │
                  Container
                       │
                    APP


                    STORAGE
                       │
                      PVC
                       │
                       PV
                       │
                 StorageClass
                       │
                      CSI
                       │
                   STORAGE


                    SECURITY
                       │
               Authentication
                       │
               Authorization
                       │
                    RBAC
                       │
                   Admission
                       │
                  Policy
```

---

# 29. Final Mental Model

Don't memorize Kubernetes as:

```text
API Server
etcd
Scheduler
Controller Manager
Kubelet
kube-proxy
CNI
CSI
Ingress
Service
...
```

Memorize it as a **city**:

```text
CITY
│
├── Administration
│   ├── API Server → Front desk
│   ├── etcd → Memory
│   ├── Scheduler → Placement planner
│   └── Controllers → Inspectors
│
├── Buildings
│   └── Nodes
│       ├── Kubelet → Building manager
│       ├── Runtime → Engine
│       └── Pods → Apartments
│
├── Roads
│   ├── CNI → Roads
│   ├── Service → Stable address
│   ├── DNS → Directory
│   └── Ingress → City gate
│
├── Utilities
│   └── Storage → Persistent storage
│
└── Security
    ├── Authentication → Who are you?
    ├── RBAC → What can you do?
    └── Policies → What is allowed?
```

## 🧠 The sentence to remember forever

> **"Kubernetes is a city where the Control Plane decides and remembers, Nodes provide the buildings, Kubelet manages the apartments, the Runtime runs the people, Services provide stable addresses, and Controllers continuously make reality match what we asked for."**

Once this mental model is clear, Kubernetes stops looking like **50 unrelated components** and starts looking like **one system doing one job: continuously keeping reality aligned with your desired state.**
